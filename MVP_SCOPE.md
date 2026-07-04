# MVP Scope

The goal of the MVP is to **validate that people will use Flirt to generate
replies** — with the least surface area needed to feel real.

> **Powerful MVP, no product limits.** During the MVP the product runs
> **without enforced usage limits** — it should feel genuinely useful, not
> crippled. Limits are *designed and metered* from day one (see COST_MODEL.md)
> but only *enforced* starting v0.3. The single exception is a high anti-abuse
> rate ceiling to protect against bots.

## In scope (v0.1 — app only, no keyboard yet)

- Paste or type the received message.
- Choose a tone: **Light Flirt, Deep Flirt, Funny, Confident, Professional**.
- Generate **3 reply suggestions** via the backend.
- Edit a suggestion inline.
- Copy a suggestion to the clipboard.
- Backend `POST /ai/replies` backed by the multi-provider AI layer.
- Local Docker environment (API + Postgres + Redis).
- Anonymous device-scoped usage (no full account required yet).

## In scope (v0.2 — keyboard MVP)

- Custom Keyboard Extension with tone buttons.
- Generate reply and **insert directly** into the host app.
- Refine actions: **Shorter, Funnier, More Direct**.
- Basic login in the container app; token shared via App Groups.

## Out of scope for MVP (deliberately deferred)

- Payments / subscriptions (StoreKit) → v0.4
- Accounts with email verification, password reset → later
- AI Dating Coach / Interest Score → Phase 6
- Screenshot + OCR ingestion → Phase 6
- Personality learning from user history → Phase 2+
- Android / web
- Team/multi-user features

## Success criteria

- A user can go from "received message" to "inserted reply" in **≤ 3 taps** once
  the keyboard is enabled.
- Median reply generation latency **< 3s**.
- Reply suggestions are on-tone and safe (pass moderation).
- Enough repeat usage to justify building payments.

## Non-goals

- Not a general chatbot.
- Not a copy/paste-only utility (that flow is explicitly rejected — see idea.md).
- Not storing message content beyond what's needed for history the user opts into.
