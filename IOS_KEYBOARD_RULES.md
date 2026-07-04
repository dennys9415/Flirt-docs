# iOS Keyboard Extension Rules

Hard constraints Apple imposes on custom keyboard extensions. Design the keyboard
target around these — most "why doesn't this work" issues trace back here.

## Network access requires "Open Access"

- By default a keyboard extension has **no network access**.
- To call the backend, the user must enable **"Allow Full Access"** for the Flirt
  keyboard in iOS Settings.
- The app must work gracefully (clear prompt to enable Full Access) when it's off.
- Never claim network works without Full Access — it won't.

## Data sharing: App Groups only

- The keyboard and container app **cannot** share data via normal storage.
- Use an **App Group** shared container for: auth token, user tone preferences,
  cached settings, usage counters.
- Configure the App Group capability on **both** targets with the same group id
  (e.g. `group.com.<org>.flirt`).

## Memory budget is tight

- Keyboard extensions run under a **strict memory limit** (much lower than the app).
- Keep the extension lightweight: no heavy image assets, no large in-memory caches,
  no bundling model code. Do AI work on the **backend**, not on-device.
- Exceeding memory → the extension is killed by the system.

## No provider secrets in the keyboard

- The keyboard must never hold OpenAI/Claude/Gemini API keys.
- It authenticates to **our** backend with a short-lived token from the App Group;
  the backend holds provider keys.

## UX / capability limits

- A keyboard cannot present arbitrary system UI; keep interactions within the
  keyboard's own view.
- Provide the standard **"next keyboard" (globe)** switch.
- Respect light/dark appearance and dynamic type.
- Text insertion uses the text document proxy (`insertText`) into the host app.
- Some secure fields (passwords) disable custom keyboards — expected, don't fight it.

## Privacy & review

- App Store review scrutinizes keyboards that request Full Access. Be explicit in
  the privacy policy about what is sent to the backend and why.
- Only send the message text and tone needed to generate a reply; no silent
  keystroke logging.

## Testing

- The keyboard **must be tested on a physical iPhone** for realistic memory and
  Full Access behavior; the simulator hides some limits.
- Test the "Full Access disabled" path explicitly.

## References

- Creating a custom keyboard: https://developer.apple.com/documentation/UIKit/creating-a-custom-keyboard
