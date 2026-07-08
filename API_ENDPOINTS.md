# API Endpoints

Base URL (local): `http://localhost:3000`
Format: JSON. Auth: `Authorization: Bearer <accessToken>` unless noted.
Shared request/response types live in `flirt-contracts`.

## Conventions

- All timestamps ISO-8601 UTC.
- Errors: `{ "error": { "code": string, "message": string } }` with proper HTTP status.
- Rate limits return `429` with `Retry-After`.

---

## Auth

### POST /auth/device
Register/identify an anonymous device (used before full account exists).
```json
// req
{ "deviceIdentifier": "…", "platform": "ios" }
// res
{ "accessToken": "…", "refreshToken": "…", "deviceId": "uuid" }
```

### POST /auth/register
Create an account; links the calling device. `username` optional
(`/^[a-z0-9_]{3,30}$/`, added v0.5).
```json
// req
{ "email": "…", "password": "…", "username": "…", "deviceIdentifier": "…" }
// res
{ "accessToken": "…", "refreshToken": "…", "deviceId": "…",
  "user": { "id": "…", "email": "…", "plan": "free" } }
```
Errors: `409 email_taken`, `409 username_taken`.

### POST /auth/login
Accepts `email` OR `username` (v0.5, additive).
```json
// req
{ "email": "…", "password": "…", "deviceIdentifier": "…" }
// or: { "username": "…", "password": "…", "deviceIdentifier": "…" }
// res — same shape as /auth/register
{ "accessToken": "…", "refreshToken": "…", "deviceId": "…",
  "user": { "id": "…", "email": "…", "plan": "free" } }
```
Errors: `401 invalid_credentials`.

### POST /auth/refresh
```json
{ "refreshToken": "…" }  →  { "accessToken": "…", "refreshToken": "…" }
```

---

## AI

### POST /ai/replies
Generate tone-based reply suggestions. **Core endpoint.**
```json
// req
{
  "message": "Hey, how was your weekend?",
  "tone": "light_flirt",
  "intent": "reply",
  "context": { "appHint": "tinder", "history": [] }
}
// res
{
  "tone": "light_flirt",
  "intent": "reply",
  "suggestions": [
    { "text": "It was good, but better if you'd been there 😏", "style": "playful" },
    { "text": "Pretty chill — what did you get up to?", "style": "curious" },
    { "text": "Weekend was solid. Yours worth stealing ideas from?", "style": "confident" }
  ],
  "provider": "openai",
  "model": "<resolved-model-id>"
}
```

### POST /ai/refine
Refine an existing suggestion.
```json
// req
{ "text": "…", "action": "shorter" }   // shorter | funnier | more_direct
// res
{ "text": "…", "style": "…" }
```

---

## Users

### GET /users/me
Returns `{ "user": {...} }`, or `{ "user": null }` for anonymous devices.

### PATCH /users/profile
Requires an account (`403 account_required` for anonymous devices).
```json
{ "displayName": "…", "personality": { "traits": ["funny","confident"] },
  "historyOptIn": true }
```

---

## AI settings (BYOK, v0.5)

Users can bring their own AI key. Requires an account. The key is stored
encrypted (AES-256-GCM); responses only ever include a masked view. When set,
`/ai/replies` uses the user's provider/key (`keySource: "user_key"`).

### PUT /users/ai-settings
```json
// req
{ "provider": "anthropic", "apiKey": "sk-ant-…", "model": "claude-opus-4-8" }
// res
{ "provider": "anthropic", "model": "claude-opus-4-8", "apiKeyMasked": "sk-a…-key" }
```
Errors: `400 unknown_provider`, `400 byok_unavailable`, `403 account_required`.

### GET /users/ai-settings
`{ "settings": { … } }` or `{ "settings": null }`.

### DELETE /users/ai-settings
`204` — reverts the user to the system provider.

---

## Usage

### GET /usage
`enforced` is `false` during the MVP (limits designed, not active).
```json
{ "plan": "free", "used": 7, "limit": 20, "enforced": false,
  "resetsAt": "2026-07-06T00:00:00Z" }
```

---

## History

### GET /history?limit=20
Only returns generations made while `historyOptIn` was on. Scoped to the
account when logged in, otherwise to the device.
```json
{ "entries": [
  { "id": "…", "message": "…", "tone": "funny",
    "suggestions": [{ "text": "…", "style": "…" }], "createdAt": "…" }
] }
```

---

## Subscriptions

### POST /subscriptions/verify
Record a StoreKit transaction and upgrade the user's plan. Requires an
account (`403 account_required` for anonymous devices).

**v0.4 note:** in `SUBSCRIPTION_VERIFY_MODE=trust_client` (local dev) the
claim is trusted — StoreKit-testing transactions can't be verified against
Apple. Real App Store Server API verification lands at v1.0 with the Apple
Developer account.
```json
// req
{ "transactionId": "…", "productId": "com.singularitybox.flirt.pro.monthly",
  "environment": "storekit_test", "expiresAt": "2026-08-05T…" }
// res
{ "plan": "pro", "status": "active", "expiresAt": "2026-08-05T…" }
```
Errors: `400 unknown_product`, `403 account_required`.

---

## Health

### GET /health → `{ "status": "ok" }`

---

## Endpoint summary

| Method | Path | Auth | Phase | Status |
|---|---|---|---|---|
| POST | /auth/device | none | v0.1 | ✅ |
| POST | /auth/register | none | v0.3 | ✅ |
| POST | /auth/login | none | v0.3 | ✅ |
| POST | /auth/refresh | none | v0.1 | ✅ |
| POST | /ai/replies | yes | v0.1 | ✅ |
| POST | /ai/refine | yes | v0.2 | ✅ |
| GET | /users/me | yes | v0.3 | ✅ |
| PATCH | /users/profile | yes | v0.3 | ✅ |
| GET | /usage | yes | v0.3 | ✅ |
| GET | /history | yes | v0.3 | ✅ |
| PUT | /users/ai-settings | yes | v0.5 | ✅ |
| GET | /users/ai-settings | yes | v0.5 | ✅ |
| DELETE | /users/ai-settings | yes | v0.5 | ✅ |
| POST | /subscriptions/verify | yes | v0.4 | ✅ (trust_client mode) |
| GET | /health | none | v0.1 | ✅ |
