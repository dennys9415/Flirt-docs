# Cost Model

Unit economics for Flirt. The product's variable cost is almost entirely AI
inference; this doc gives the math so we always know what a user costs us.

> **MVP philosophy:** during the MVP we run **without enforced limits** — the
> product should feel powerful and unrestricted. But every request is metered
> from day one (`usage_events` + Redis counters), so when we flip limits on
> (v0.3+) we already have real data on what users actually consume.

## Cost per generation (the core unit)

One `POST /ai/replies` call ≈:

| Component | Tokens (est.) |
|---|---|
| System prompt (tones, rules, schema) | ~400 in |
| Received message + context | ~100–300 in |
| 3 suggestions (JSON) | ~150–250 out |

Round numbers for modeling: **~600 input / ~200 output tokens per generation**.

## Provider pricing (verify before launch — prices change)

Claude prices verified 2026-07 from Anthropic docs. OpenAI/Gemini rows must be
filled from their current price pages before doing final math — do **not**
trust remembered numbers.

| Provider / model | Input $/1M | Output $/1M | Cost per generation (600 in / 200 out) |
|---|---|---|---|
| Claude Haiku 4.5 | $1.00 | $5.00 | ~$0.0016 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | ~$0.0048 |
| Claude Opus 4.8 | $5.00 | $25.00 | ~$0.0080 |
| OpenAI (small model) | _fill in_ | _fill in_ | _fill in_ |
| OpenAI (flagship) | _fill in_ | _fill in_ | _fill in_ |
| Gemini (small model) | _fill in_ | _fill in_ | _fill in_ |

**This is exactly why the multi-provider layer exists:** route cheap models to
casual usage and premium models to paying users, per config — no code changes.

## What a user costs per month (scenarios)

Using a mid-range ~$0.005/generation:

| User type | Generations/day | Cost/month |
|---|---|---|
| Casual | 5 | ~$0.75 |
| Active | 20 | ~$3.00 |
| Heavy | 60 | ~$9.00 |
| Abusive (bot/scraper) | 500+ | $75+ ⚠️ |

Refine calls (`/ai/refine`) are ~half a generation (shorter prompt, 1 output).

## Revenue side (when payments ship, v0.4)

- Apple takes **30%** of subscription revenue (15% under the Small Business
  Program if annual revenue < $1M — we should qualify at launch).
- Example: $9.99/month Pro → **$8.49 net** (15%) or $6.99 net (30%).
- A heavy user on a mid-range model (~$9/mo) roughly **breaks even** at $9.99;
  on a cheap model (~$3/mo) margins are healthy. Model routing per tier is the
  main margin lever.

## Break-even guardrails (design now, enforce later)

| Lever | MVP (now) | Post-MVP (v0.3+) |
|---|---|---|
| Free tier limit | None enforced — metered only | e.g. 20 generations/day |
| Rate limiting | Anti-abuse only (very high ceiling, e.g. 100/hour) | Per-plan |
| Model routing | Best quality for everyone | Cheap model free / premium model paid |
| Refine actions | Unlimited | Count as ½ generation |

> Even in "no limits" mode, keep **one anti-abuse rate limit** (absurdly high
> for a human, low for a bot). That's not a product limitation; it's protection
> against a single bad actor burning the budget.

## Monitoring

- Log `provider`, `model`, `latency_ms` and token usage per request
  (`reply_requests` table) → daily cost dashboard = sum(tokens × price).
- Alert if daily spend exceeds a threshold or one device exceeds N generations.
