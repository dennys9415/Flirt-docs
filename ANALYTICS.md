# Analytics

Define success **before** shipping v0.1 — otherwise we can't validate anything.
Since the MVP runs **without enforced limits**, analytics is also how we learn
what real usage looks like before designing the paid tiers.

## North Star

**Weekly inserted replies per active user** — a reply the user actually sent
(inserted or copied) is the moment Flirt delivered value.

## Event schema

All events carry: `device_id`, `user_id?`, `timestamp`, `app_version`,
`surface` (`app` | `keyboard`).

### Activation funnel
| Event | When |
|---|---|
| `app_installed` | First launch |
| `onboarding_completed` | Finished intro |
| `keyboard_enabled` | Flirt keyboard added in iOS Settings (detected) |
| `full_access_granted` | Full Access turned on |
| `first_generation` | First successful `/ai/replies` |
| `first_insert` | First suggestion inserted/copied |

### Core usage
| Event | Properties |
|---|---|
| `reply_generated` | `tone`, `latency_ms`, `provider`, `model`, `surface` |
| `suggestion_inserted` | `tone`, `position` (1–3), `surface` |
| `suggestion_copied` | same |
| `suggestion_edited` | same |
| `refine_used` | `action` (shorter/funnier/more_direct) |
| `generation_failed` | `error_code`, `latency_ms` |
| `moderation_blocked` | `direction` (input/output) |

### Quality signals
- **Insert rate** = `suggestion_inserted` / `reply_generated` — the single best
  quality proxy. If people generate but don't insert, suggestions are bad.
- **Position bias**: which of the 3 suggestions wins → informs prompt tuning.
- **Refine rate** per tone: high refine rate on a tone = tone is miscalibrated.

## KPIs & validation targets (MVP)

| KPI | Definition | Target to justify continuing |
|---|---|---|
| Activation | install → `first_insert` | > 30% |
| Keyboard adoption | installs → `keyboard_enabled` | > 40% (v0.2) |
| Full Access grant | keyboard_enabled → full_access | > 70% |
| Insert rate | inserted / generated | > 45% |
| D7 retention | active day 7 / installs | > 15% |
| Latency p50 | reply generation | < 3s |
| Weekly generations per active user | — | > 15 |

These numbers also feed COST_MODEL.md: real generations/user/day tells us where
to draw the free-tier line later — with data, not guesses.

## Implementation notes

- **Backend-first**: usage events already land in `usage_events`; most metrics
  above are derivable server-side. Prefer that over heavy client SDKs for MVP.
- Client events (onboarding, keyboard enablement) need a lightweight client
  logger → `POST /events` batch endpoint (add to API when implementing).
- **Privacy:** analytics events must never contain message text or suggestion
  text. Metadata only. Declare analytics in App Store privacy labels
  (see APP_STORE_COMPLIANCE.md).
- Keyboard surface: buffer events in the App Group container and flush from the
  keyboard's network call (Full Access required anyway for generation).
