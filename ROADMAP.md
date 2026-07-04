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

## v0.2 — Keyboard MVP
- Custom Keyboard Extension with tone buttons + Insert.
- Refine actions: Shorter / Funnier / More Direct (`POST /ai/refine`).
- App Groups sharing (token + settings) between app and keyboard.
- Full Access prompt + graceful offline state.
- **Exit criteria:** reply inserted into a real host app in ≤ 3 taps on a device.

## v0.3 — Users & limits
- Email login (`/auth/login`, `/auth/refresh`), profile (`/users/*`).
- Opt-in history; usage metering (`/usage`), Redis rate limiting.
- Free vs Pro limits enforced server-side.
- **Exit criteria:** per-user limits and history work; abuse is rate-limited.

## v0.4 — Payments
- StoreKit subscription (monthly), free trial, upgrade screen.
- `POST /subscriptions/verify` server-side receipt verification.
- Plan gating: free (limited) / pro (unlimited) / premium (coach).
- **Exit criteria:** a user can subscribe and unlock Pro, verified on backend.

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
