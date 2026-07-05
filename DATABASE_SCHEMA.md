# Database Schema (PostgreSQL)

Target: PostgreSQL 16. IDs are UUID v4. Timestamps are `timestamptz` (UTC).

**Schema is owned by Flyway** (Signalix-style): versioned SQL migrations in
`flirt-api/migrations/` (`V1__init.sql`, `V2__*.sql`, …), executed by the
Flyway container in `flirt-infra` before the API starts. Application code never
alters the schema. Never edit an applied migration — add a new version.
Data access is raw SQL via `pg` (no ORM).

## Entity overview

```text
users ──1:N── devices
  │
  ├──1:N── subscriptions
  ├──1:N── usage_events
  └──1:N── reply_requests ──1:N── reply_suggestions
```

## Tables

### users
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| email | text | nullable until account created; unique when present |
| password_hash | text | bcrypt; nullable (V2) |
| display_name | text | nullable |
| plan | text | `free` \| `pro` \| `premium`, default `free` |
| personality | jsonb | tone prefs, style hints (nullable) |
| history_opt_in | boolean | default false — gates content persistence (V2) |
| created_at | timestamptz | default now() |
| updated_at | timestamptz | |

### devices
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | FK → users.id (nullable for anonymous device) |
| device_identifier | text | vendor id / installation id, unique |
| platform | text | `ios` |
| created_at | timestamptz | |

### subscriptions
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | FK → users.id |
| plan | text | `pro` \| `premium` |
| status | text | `active` \| `expired` \| `grace` \| `canceled` |
| store | text | `app_store` |
| original_transaction_id | text | Apple transaction id |
| expires_at | timestamptz | |
| created_at | timestamptz | |

### reply_requests
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | FK → users.id (nullable) |
| device_id | uuid | FK → devices.id |
| tone | text | `light_flirt` \| `deep_flirt` \| `funny` \| `confident` \| `professional` |
| intent | text | `reply` \| `rewrite` \| `refine` |
| input_message | text | the received message (retained only if history opted in) |
| provider | text | `openai` \| `claude` \| `gemini` |
| model | text | resolved model id |
| latency_ms | int | |
| created_at | timestamptz | |

### reply_suggestions
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| request_id | uuid | FK → reply_requests.id |
| text | text | suggestion body |
| style | text | freeform style label (e.g. `playful`) |
| position | int | display order |

### usage_events
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | FK → users.id (nullable) |
| device_id | uuid | FK → devices.id |
| kind | text | `reply_generate` \| `refine` |
| created_at | timestamptz | |

> Real-time counters for rate limiting live in **Redis**; `usage_events` is the
> durable audit trail used for analytics and plan enforcement.

## Privacy

- `input_message` and suggestion text are stored **only** when the user opts into
  history. Otherwise the request row keeps metadata (tone, provider, latency) but
  not content.
- Provide a delete path that cascades from `users` → all child rows.
