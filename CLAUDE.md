# CLAUDE.md — Working conventions

Guidance for Claude and contributors working on the Flirt project.

## Language
- **All code, comments, docs, commit messages, and identifiers are in English.**
- The product owner communicates in Spanish in chat; deliverables stay English.

## Product in one line
An iOS app + custom keyboard that generates tone-based reply suggestions for
dating/chat apps, powered by a backend multi-provider AI layer.

## Repos
| Repo | Stack | Role |
|---|---|---|
| `flirt-ios` | Swift, SwiftUI, Keyboard Extension, StoreKit | App + keyboard |
| `flirt-api` | NestJS, PostgreSQL, Redis, Docker | Backend + AI layer |
| `flirt-contracts` | TypeScript, JSON Schema | Shared request/response types |
| `flirt-infra` | Docker Compose → AWS | Infrastructure |
| `flirt-docs` | Markdown | Specs (this repo) |

## Non-negotiable decisions
1. **Single App Store app**, two targets (app + keyboard). Share state via **App Groups**, not the network.
2. **Multi-provider AI layer** (OpenAI / Claude / Gemini) behind one `AIProvider` interface; provider/model are **config-driven**.
3. **Structured JSON outputs only** — the client never parses free-form text. Schema lives in `flirt-contracts`.
4. **Secrets stay server-side.** The keyboard never holds provider API keys; it uses a short-lived token from the App Group.
5. **Backend owns limits, usage, moderation, and payments verification.**

## Build order
Follow ROADMAP.md: v0.1 (app + backend reply flow) → v0.2 (keyboard) → v0.3
(users/limits) → v0.4 (payments) → v1.0 (production). Validate before expanding.

## Conventions
- REST endpoints and payloads: see API_ENDPOINTS.md (keep it in sync with `flirt-contracts`).
- **Signalix is the reference architecture** (~/SingularityBox/Signalix): Flyway
  versioned SQL migrations in `flirt-api/migrations/` (`V<n>__name.sql`), raw
  SQL via a global `DbService` (`pg` Pool, no ORM), and infra centralized in
  `flirt-infra` (compose + `env/*.env` + `scripts/up|down|migrate|logs.sh`).
- DB changes = new Flyway migration (never edit an applied one) + update DATABASE_SCHEMA.md.
- Keyboard work must respect IOS_KEYBOARD_RULES.md (Full Access, memory, App Groups).
- Prompts/tones/safety: AI_PROMPTS.md is the source of truth.

## When in doubt
- Prefer the smallest change that ships a working slice of the current phase.
- Keep the keyboard extension lightweight; push work to the backend.
- Update the relevant doc in this repo when a decision changes.
