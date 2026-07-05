# Handoff — Project Status

> Last updated: **2026-07-05**. Update this file at the end of every working
> session (Signalix convention).

## Where the project stands

**v0.1, v0.2 (Keyboard MVP) and v0.3 (Users & History) are COMPLETE** and
verified in the simulator. The keyboard generates real Gemini replies and
inserts into host apps; the app now has onboarding, email accounts
(register/login with device linking), opt-in history, and usage display.
62 automated tests (42 unit + 20 e2e) green in CI.

v0.3 highlights: limits machinery is complete but **enforced=false**
(powerful-MVP); history content is only persisted with explicit
`historyOptIn`; anonymous devices keep working without an account.

v0.2 architecture: `Shared/` sources compiled into both targets; App Group
(`group.com.singularitybox.flirt`) carries tokens + selected tone; the app
provisions the device identity at launch (`APIClient.warmUp()`); the keyboard
never registers its own device. Keyboard flow is clipboard-based (copy the
received message → tap tone → insert).

| Repo | Status | Detail |
|---|---|---|
| `flirt-docs` | ✅ Complete | 13 docs: architecture, scope, DB, API, keyboard rules, prompts, roadmap, compliance, cost model, analytics, testing, conventions |
| `flirt-api` | ✅ v0.1 + tests/CI | NestJS · raw SQL via `DbService` (pg Pool, no ORM) · Flyway migrations · multi-provider AI layer (fake/openai/anthropic/gemini) · device auth JWT · metering ON, limits OFF · **36 unit + 9 e2e tests, GitHub Actions CI** |
| `flirt-infra` | ✅ Working | Signalix-style: compose (postgres → flyway → redis → api → adminer) + `env/*.env` + `scripts/up\|down\|migrate\|logs.sh` |
| `flirt-ios` | ✅ v0.2 working | App + **Keyboard Extension** (XcodeGen, 2 targets + Shared/) · App Group token sharing · clipboard-based keyboard flow verified in simulator |
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

Agreed sequencing: finish everything buildable without an Apple Developer
account; enroll ($99) only at the end for TestFlight/App Store.

1. **v0.4 — StoreKit local**: subscription products via StoreKit
   Configuration file (testable in simulator without App Store Connect),
   upgrade screen, `POST /subscriptions/verify` backend stub.
2. **Deploy the backend publicly** (Railway/Fly) with HTTPS — prerequisite
   for any real-device testing.
3. **`flirt-contracts`** — extract shared JSON schemas.
4. End of project (needs Apple account): physical iPhone keyboard test,
   shared Keychain migration, TestFlight, App Store submission.

## Testing (added 2026-07-05)

- `npm test` — 36 unit tests (providers, prompts, factory, usage fail-open,
  AiService 502 mapping + privacy assertion, AuthService).
- `npm run test:e2e` — 9 supertest tests against real Postgres/Redis with the
  fake provider (start `Flirt-infra/scripts/up.sh` first).
- CI: `.github/workflows/ci.yml` — build + unit + migrations (psql) + e2e on
  every push/PR with service containers.

## v0.2 keyboard debt (carry into v1.0 gate)

- Tokens in App Group UserDefaults (not encrypted) — migrate to shared
  Keychain access group before TestFlight.
- Keyboard is suggestion-only (no QWERTY) — evaluate App Review risk; may need
  minimal typing support.
- Full Access on the simulator was granted via prefs; the real Settings flow
  must be walked through on a device.
