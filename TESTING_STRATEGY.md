# Testing Strategy

Lightweight but deliberate: automate where it's cheap (API), checklist where
automation is impossible (keyboard on a physical device).

## Backend (flirt-api) — automated

### Unit tests
- **AI Provider Layer is the priority.** Test each adapter (OpenAI/Claude/
  Gemini) against mocked HTTP responses: happy path, malformed JSON from the
  model, provider timeout, provider 429/5xx.
- Tone/prompt builder: given tone + message → expected prompt structure.
- Moderation gate: unsafe input/output is blocked or replaced.
- Usage metering: every generation writes a `usage_events` row and increments
  the Redis counter (even though limits aren't enforced in MVP).

### Integration tests
- `POST /ai/replies` end-to-end with a **fake provider** (deterministic
  fixture): returns exactly 3 suggestions, valid schema, provider/model echoed.
- Auth: `POST /auth/device` issues tokens; protected routes reject bad tokens.
- Anti-abuse rate limit returns 429 with `Retry-After` past the ceiling.
- Run against Docker Compose (Postgres + Redis) in CI (GitHub Actions).

### Contract tests
- Validate API responses against the JSON schemas in `flirt-contracts`.
  This is what protects old installed iOS versions when the API evolves —
  breaking the schema must break CI.

### What NOT to automate
- Real provider calls in CI (cost + flakiness). Keep one manual smoke script
  (`scripts/smoke-live.sh`) that hits each real provider before releases.

## iOS app — automated where cheap
- Unit tests for view models: state transitions (loading → suggestions →
  error), clipboard copy, edit flow.
- Snapshot tests optional; skip for MVP.

## Keyboard extension — manual checklist (physical iPhone required)

The simulator hides memory limits and Full Access behavior. Before every
TestFlight build, run on a real device:

- [ ] Keyboard appears in Settings and can be added
- [ ] Basic typing + globe switch work with **Full Access OFF**
- [ ] Full Access prompt appears and explains why (compliance)
- [ ] With Full Access ON: generate → 3 suggestions → Insert into iMessage,
      WhatsApp, and Instagram
- [ ] Refine actions (Shorter/Funnier/More Direct) work
- [ ] Airplane mode: graceful error, no crash
- [ ] Password field: custom keyboard suppressed by iOS (expected — verify no crash on return)
- [ ] Light/dark mode render correctly
- [ ] Memory: long session (20+ generations) without the extension being killed
- [ ] Token expiry: keyboard recovers by refreshing via App Group

## Release gate (per version)
1. CI green (unit + integration + contract).
2. Live-provider smoke script passes.
3. Keyboard manual checklist on a physical device.
4. APP_STORE_COMPLIANCE.md checklist re-verified before store submissions.
