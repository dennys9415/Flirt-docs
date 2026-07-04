# AI Prompts

How Flirt turns a received message + tone into structured reply suggestions.
The AI layer is **multi-provider** (OpenAI / Claude / Gemini); prompts are written
provider-agnostically and each adapter maps them to that vendor's API.

## Contract: structured output

Every generation must return JSON matching this shape (defined in `flirt-contracts`):

```json
{
  "tone": "light_flirt",
  "intent": "reply",
  "suggestions": [
    { "text": "string", "style": "string" }
  ]
}
```

- Exactly **3** suggestions for `intent: reply`.
- No markdown, no preamble, no explanation — JSON only.
- Enforce via provider-native structured output / JSON schema where available
  (OpenAI structured outputs, Claude tool use, Gemini response schema).

## Tones

| Tone key | Intent | Guidance |
|---|---|---|
| `light_flirt` | Playful, warm, low-risk. A little charm, never crude. |
| `deep_flirt` | Bolder, more direct interest. Confident, still respectful. |
| `funny` | Lead with humor; witty, teasing, light. |
| `confident` | Self-assured, concise, no neediness. |
| `professional` | Friendly-professional (Slack/LinkedIn/Gmail). No flirtation. |

## System prompt (template)

```text
You are Flirt, an assistant that writes short, natural reply suggestions for
messaging and dating apps. You will be given a received message and a target tone.

Rules:
- Write replies as the USER would send them (first person, casual).
- Match the requested tone exactly.
- Keep each reply concise (1–2 sentences) and easy to send as-is.
- Sound human: no clichés, no pickup-artist lines, no over-explaining.
- Never be crude, demeaning, coercive, or sexually explicit.
- If the received message signals discomfort, disinterest, or is about a minor or
  anything unsafe, refuse by returning a single respectful, de-escalating suggestion.
- Return ONLY JSON matching the provided schema.
```

## User prompt (template)

```text
Received message: "{{message}}"
Target tone: {{tone}}
Intent: {{intent}}
Context: app={{appHint}}, prior_turns={{history_count}}
Personality hints: {{personality_traits}}

Produce {{count}} reply suggestions.
```

## Refine actions (`POST /ai/refine`)

| Action | Instruction |
|---|---|
| `shorter` | Rewrite the reply to be noticeably shorter, same tone/meaning. |
| `funnier` | Add light humor without changing the core intent. |
| `more_direct` | Make the interest/ask clearer and more confident. |

## Safety & moderation

- Run moderation on **input** and **output**. Block/replace anything sexual about
  minors, harassment, threats, or self-harm content.
- Prefer refusal-with-alternative over hard error so the UX stays graceful.
- Log the provider + model used per request for auditing (see DATABASE_SCHEMA.md).

## Model configuration

- Provider and model are **config-driven** (env / plan tier), never hardcoded in
  call sites. The `AIProvider` interface exposes `generateReplies(input)` and
  `refine(input)`; adapters resolve the concrete model id at request time and
  return it in the response for observability.
