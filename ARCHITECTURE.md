# Architecture

## Overview

Flirt is an iOS app that helps users craft replies in dating and chat apps
(Tinder, Bumble, Hinge, WhatsApp, Instagram, etc.) using AI. The core of the
product is a **custom keyboard extension** that generates tone-based replies and
inserts them directly into whatever app the user is typing in — no copy/paste
round-trips.

## Components

```text
┌──────────────────────────────┐
│         Flirt App            │   Swift + SwiftUI
│  Login · Subscription        │
│  Settings · History          │
│  Personality profiles        │
└──────────────┬───────────────┘
               │  App Groups (shared container)
               │  - auth token
               │  - user settings / tones
               │  - usage counters
┌──────────────▼───────────────┐
│      Flirt Keyboard          │   Custom Keyboard Extension
│  Tone buttons                │
│  Generate reply              │
│  Insert / Shorter / Funnier  │
└──────────────┬───────────────┘
               │  HTTPS (JSON)
┌──────────────▼───────────────┐
│        Flirt API             │   NestJS
│  /auth  /users  /ai  /usage  │
│  /subscriptions              │
│  ┌────────────────────────┐  │
│  │  AI Provider Layer     │  │  swappable
│  │  OpenAI│Claude│Gemini  │  │
│  └────────────────────────┘  │
└───────┬───────────────┬──────┘
        │               │
   ┌────▼────┐     ┌────▼────┐
   │Postgres │     │  Redis  │   cache · rate limit · usage
   └─────────┘     └─────────┘
```

## Data flow: generating a reply

1. User taps a tone (e.g. **Light Flirt**) in the keyboard extension.
2. Keyboard reads the auth token + user settings from the App Group container.
3. Keyboard sends `POST /ai/replies` with `{ message, tone, intent, context }`.
4. API checks auth, rate limit and remaining usage (Redis).
5. AI Provider Layer selects the configured provider/model and requests a
   **structured JSON** response (list of suggestions).
6. API persists usage + optional history (Postgres), returns suggestions.
7. Keyboard renders suggestions; user taps **Insert** to place text in the host app.

## Key architectural decisions

- **Single App Store app, two targets.** The keyboard is an extension inside the
  container app. Shared state travels through **App Groups**, never through the
  network between the two.
- **Multi-provider AI layer.** A single `AIProvider` interface with adapters per
  vendor (OpenAI / Claude / Gemini). Provider + model are config-driven so we can
  switch per environment, plan tier, or request without touching call sites.
- **Structured outputs only.** The AI must return JSON matching a schema defined
  in `flirt-contracts`. The app never parses free-form text.
- **Backend owns cost & limits.** API keys, rate limiting, usage metering and
  plan enforcement live server-side. The keyboard never holds provider keys.
- **Stateless keyboard.** The extension is memory-constrained (see
  IOS_KEYBOARD_RULES.md); it caches nothing sensitive and does minimal work.

## Environments

| Env | iOS | Backend | DB / Cache |
|---|---|---|---|
| Local | Xcode + simulator/device | Docker via `flirt-infra` (`scripts/up.sh`) | Docker Postgres + Redis + adminer |
| Production | TestFlight → App Store | AWS (ECS/App Runner) | RDS Postgres + ElastiCache Redis |

**Schema migrations:** Flyway (versioned SQL in `flirt-api/migrations/`), run
by a Flyway container before the API starts — the API depends on
`flyway: service_completed_successfully`. Data access is raw SQL through a
`DbService` (`pg` Pool); no ORM. Pattern adopted from the Signalix project.

## Security notes

- Auth via JWT (short-lived access + refresh).
- Provider API keys stored in backend secrets (env locally, Secrets Manager in prod).
- All traffic over HTTPS/TLS.
- Content moderation on AI input/output before returning to the client.
