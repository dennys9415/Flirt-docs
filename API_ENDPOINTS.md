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

### POST /auth/login
```json
// req
{ "email": "…", "password": "…" }
// res
{ "accessToken": "…", "refreshToken": "…", "user": { "id": "…", "plan": "free" } }
```

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
Returns the current user/profile and plan.

### PATCH /users/profile
```json
{ "displayName": "…", "personality": { "traits": ["funny","confident"] } }
```

---

## Usage

### GET /usage
```json
{ "plan": "free", "used": 7, "limit": 20, "resetsAt": "2026-07-04T00:00:00Z" }
```

---

## Subscriptions

### POST /subscriptions/verify
Verify an App Store transaction and update the user's plan.
```json
// req
{ "transactionId": "…", "receipt": "…" }
// res
{ "plan": "pro", "status": "active", "expiresAt": "…" }
```

---

## Health

### GET /health → `{ "status": "ok" }`

---

## Endpoint summary

| Method | Path | Auth | Phase |
|---|---|---|---|
| POST | /auth/device | none | v0.1 |
| POST | /auth/login | none | v0.3 |
| POST | /auth/refresh | none | v0.3 |
| POST | /ai/replies | yes | v0.1 |
| POST | /ai/refine | yes | v0.2 |
| GET | /users/me | yes | v0.3 |
| PATCH | /users/profile | yes | v0.3 |
| GET | /usage | yes | v0.3 |
| POST | /subscriptions/verify | yes | v0.4 |
| GET | /health | none | v0.1 |
