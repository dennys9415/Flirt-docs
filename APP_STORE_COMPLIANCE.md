# App Store Compliance

Flirt combines three things Apple reviews strictly: a **keyboard with Full
Access**, **AI-generated content**, and a **dating context**. A rejection can
cost weeks — design for review from day one instead of patching at v1.0.

## Risk register

### 1. Keyboard "Allow Full Access" (highest scrutiny)
- A keyboard extension needs Full Access to reach the network. Apple's
  guidelines require keyboards to be usable **without** Full Access and to
  collect only data needed for core functionality.
- **Mitigations:**
  - The keyboard must function as a basic keyboard (type + insert) with Full
    Access off, and show a friendly prompt explaining why Full Access unlocks AI.
  - Send only: the message being replied to, tone, and auth token. **No
    keystroke logging, no background collection, ever.**
  - State exactly this in the privacy policy and in the App Store privacy labels.

### 2. AI-generated content moderation
- Apple requires apps with AI-generated content to have content filtering and
  a way to report offensive content.
- **Mitigations:**
  - Moderation on input and output server-side (see AI_PROMPTS.md).
  - A "report suggestion" affordance in the app (can be minimal in MVP).
  - Refuse gracefully on unsafe topics (minors, harassment, coercion).

### 3. Age rating
- Dating/flirting content ⇒ expect a **17+ rating**. Declare it honestly in
  App Store Connect; underrating is a common rejection cause.

### 4. Privacy "nutrition labels" + policy
- Declare in App Store Connect: message content sent to our servers (App
  Functionality), identifiers (device id), usage data (analytics).
- Privacy policy URL is **required** before TestFlight external testing.
- Provide account/data deletion (Apple requires in-app account deletion if the
  app supports account creation) — plan for it in v0.3.

### 5. Payments (v0.4)
- All digital subscriptions must go through **StoreKit/IAP** — never link out
  to external payment for unlockable features.
- Show price, term, and auto-renewal disclosure on the upgrade screen.

### 6. Name & branding
- "Flirt" alone is likely taken/too generic on the App Store. Prepare
  candidates like "Flirt AI — Reply Keyboard" (name ≤30 chars + subtitle).
  Check availability in App Store Connect **before** investing in branding.

## Review-preparation checklist (before first submission)

- [ ] Keyboard works (basic typing) with Full Access disabled
- [ ] Full Access prompt explains data usage in plain language
- [ ] Privacy policy live at a public URL; labels filled in App Store Connect
- [ ] 17+ age rating declared
- [ ] Moderation active on AI input/output + report mechanism
- [ ] Demo account + review notes for the App Review team (explain the keyboard
      flow — reviewers often miss how to enable keyboards)
- [ ] No references to external payments
- [ ] App name/subtitle verified available

## References
- App Review Guidelines: https://developer.apple.com/app-store/review/guidelines/
- Custom keyboards guideline section: 4.4.1 (extensions)
- Privacy labels: https://developer.apple.com/app-store/app-privacy-details/
