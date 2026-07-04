# Handoff — Project Status

> Last updated: **2026-07-03**. Update this file at the end of every working
> session (Signalix convention).

## Where the project stands

**v0.1 (Local MVP) is COMPLETE and verified end-to-end**: iOS app in the
simulator → local API → real Gemini generation → metered in Postgres.

| Repo | Status | Detail |
|---|---|---|
| `flirt-docs` | ✅ Complete | 13 docs: architecture, scope, DB, API, keyboard rules, prompts, roadmap, compliance, cost model, analytics, testing, conventions |
| `flirt-api` | ✅ v0.1 working | NestJS · raw SQL via `DbService` (pg Pool, no ORM) · Flyway migrations · multi-provider AI layer (fake/openai/anthropic/gemini) · device auth JWT · metering ON, limits OFF |
| `flirt-infra` | ✅ Working | Signalix-style: compose (postgres → flyway → redis → api → adminer) + `env/*.env` + `scripts/up\|down\|migrate\|logs.sh` |
| `flirt-ios` | ✅ Phase 1 working | SwiftUI via XcodeGen: paste → 5 tones → 3 real suggestions → copy/edit/refine · Keychain tokens · transparent device auth |
| `flirt-contracts` | ⬜ Empty | Extract from API once endpoints stabilize |

## Key decisions locked in

1. **Signalix (~/SingularityBox/Signalix) is the reference architecture** —
   Flyway versioned SQL migrations (`V<n>__name.sql` in `flirt-api/migrations/`),
   raw `pg` (no ORM), infra centralized in `flirt-infra`.
2. **Multi-provider AI layer** — config-driven (`AI_PROVIDER` env var);
   currently running **Gemini free tier** (`gemini-flash-latest`) with
   structured JSON output (`responseJsonSchema`).
3. **Powerful MVP, no limits** — `ENFORCE_PLAN_LIMITS=false`; everything
   metered (`usage_events` + Redis) but unrestricted. Only an anti-abuse
   ceiling (100 req/h/device). Limits activate in v0.3 using real usage data.
4. **Privacy by design** — message text is never persisted
   (`input_message` = NULL until history opt-in ships in v0.3).
5. **All code/docs in English**; the product owner communicates in Spanish.

## How to run everything

```bash
# Backend stack (Postgres + Flyway migrate + Redis + API + adminer)
cd Flirt-infra && ./scripts/up.sh          # API at :3000, adminer at :8080

# iOS app
cd Flirt-ios && xcodegen generate          # if project.yml changed
open Flirt.xcodeproj                        # run on iPhone simulator
# (simulator reaches the API at http://localhost:3000 automatically)
```

Gemini is configured in `Flirt-infra/env/api.env` (gitignored). To switch
providers: edit `AI_PROVIDER` / `AI_MODEL` / key, then
`docker compose up -d api`.

## Known state / gotchas

- `Flirt-infra/env/api.env` holds the real Gemini API key — never commit
  (already gitignored). Rotate at https://aistudio.google.com/apikey if leaked.
- Docker image caching: after changing API code, rebuild with
  `docker compose build api` (a `.dockerignore` is in place).
- Xcode dev dir was switched to full Xcode
  (`sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`).
- No automated tests yet — TESTING_STRATEGY.md defines what to build.

## Next steps (in rough priority order)

1. **v0.2 — Keyboard Extension** (the differentiator): custom keyboard target,
   App Groups (`group.com.singularitybox.flirt`) sharing tokens/settings,
   tone buttons + insert. **Requires a physical iPhone to test properly**
   (see IOS_KEYBOARD_RULES.md).
2. **Automated tests** for the API (provider adapters with mocks, contract
   tests) per TESTING_STRATEGY.md.
3. **`flirt-contracts`** — extract shared JSON schemas from the API.
4. App polish: onboarding, local history, better error states.
5. v0.3 groundwork: email auth, opt-in history, usage endpoint.
