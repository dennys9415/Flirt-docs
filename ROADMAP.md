# Roadmap

Phased delivery. Each version is shippable/testable on its own. Don't build
everything at once — validate the reply flow first, then the keyboard, then money.

## v0.1 — Local MVP (app only) ✅ DONE 2026-07-03
- ✅ Normal iOS app: paste message → choose tone → generate 3 replies → edit → copy
  (plus refine actions, ahead of schedule).
- ✅ Backend `POST /ai/replies` with the multi-provider AI layer (running Gemini
  free tier; OpenAI/Anthropic adapters ready).
- ✅ `POST /auth/device` for anonymous device identity.
- ✅ Docker Compose via `flirt-infra`: API + Postgres + **Flyway** + Redis + adminer.
- ✅ **Exit criteria met:** end-to-end reply generation verified in the simulator.

## v0.2 — Keyboard MVP ✅ DONE 2026-07-04 (simulator)
- ✅ Custom Keyboard Extension with tone buttons + Insert (clipboard-based flow).
- ✅ App Groups sharing (token + selected tone) between app and keyboard;
  app provisions the device identity at launch (`warmUp()`).
- ✅ Full Access prompt + "open the app first" + error states.
- ✅ Reply inserted into a host app (Reminders) in ≤ 3 taps — verified in simulator.
- ⬜ Pending: verification on a **physical iPhone** (memory budget, real Full
  Access flow) and moving tokens from App Group UserDefaults to a shared Keychain.

## v0.3 — Users & limits — backend ✅ DONE 2026-07-05
- ✅ Email register/login (bcrypt), device linking, profile (`/users/*`).
- ✅ Opt-in history (`/history`; content persisted only with `historyOptIn`).
- ✅ `GET /usage` (plan, used today, limit, resetsAt) with Redis counter +
  Postgres fallback.
- ✅ Free-plan limit machinery complete but **enforced=false** (powerful-MVP
  decision — flips on via `ENFORCE_PLAN_LIMITS`).
- ⬜ iOS: account/settings UI, history screen, onboarding.

## v0.4 — Payments ✅ DONE 2026-07-05 (local StoreKit)
- ✅ StoreKit 2 subscriptions (Pro $9.99 / Premium $19.99 monthly) via local
  StoreKit Configuration — testable in the simulator, no App Store Connect.
- ✅ Paywall (upgrade screen) + restore purchases + Settings integration.
- ✅ `POST /subscriptions/verify` records the transaction and upgrades the
  plan (`trust_client` mode; V3 migration).
- ⬜ Deferred to v1.0 (needs Apple Developer account): App Store Server API
  verification, sandbox testing, free trial offer.

## v1.0 — Production
- TestFlight → App Store submission.
- AWS deploy (ECS/App Runner, RDS, ElastiCache, Secrets Manager).
- Monitoring, crash reporting, analytics.
- Privacy Policy + Terms; security review; CI/CD (GitHub Actions).
- **Exit criteria:** public release with monitoring and legal in place.

## Phase 6 — Advanced (post-1.0 differentiators)
- Screenshot + OCR ingestion of conversations.
- **AI Dating Coach**: full-conversation analysis, Interest Score, what to say next.
- Personality learning from user history.
- Professional mode for Slack / LinkedIn / Gmail.

## Repo sequencing
1. `flirt-docs` (this) — specs. ✅ in progress
2. `flirt-api` — backend + AI layer (v0.1).
3. `flirt-contracts` — shared types, extracted as the API stabilizes.
4. `flirt-ios` — app (v0.1), then keyboard (v0.2).
5. `flirt-infra` — formalize Docker → AWS around v1.0.
