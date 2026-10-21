# Product Prompt: FaceKey — iOS-Inspired, Cross-Platform Laptop Unlock

Build **FaceKey** (working name): a premium Windows 11 laptop security experience with companion apps for **both Android phones and iPhones**. The visual language should be inspired by the clarity and refinement of iOS, while the phone authentication must use each platform's official APIs. FaceKey is an independent product; do not use Apple's logo or imply it is an official Apple product.

## Product goal

Let a user authenticate to their laptop using **laptop-side face recognition only** or laptop fingerprint authentication when the laptop has compatible fingerprint hardware. Face recognition must run only on the laptop through a supported Windows Hello-compatible infrared camera; never use face recognition on the phone to approve a FaceKey unlock.

The companion app supports Android and iPhone. Phone approval must use **fingerprint authentication only** where that capability exists: Android fingerprint authentication on supported Android phones, and Touch ID on iPhones that have Touch ID. Do not invoke Face ID on an iPhone or face unlock on Android for FaceKey approvals. On iPhones without Touch ID, provide an explicit system device-passcode fallback if enabled and supported; otherwise clearly mark fingerprint approval as unavailable on that phone. Android devices must meet the selected strong biometric security policy and expose a suitable fingerprint authenticator.

On Windows laptops, call the fingerprint method **Windows Hello fingerprint**, not Touch ID. Enable it only when a compatible fingerprint reader and driver are actually available. On any supported platform, biometric templates and images stay on the device and are never transmitted; FaceKey receives only a cryptographically bound approval result.

## Design direction

- Premium iOS-inspired visual system: restrained, calm, carefully layered glass-like panels, rounded cards, generous whitespace, subtle borders, soft shadows, native platform typography and coherent icons. Do not make it look like a generic neon hacker dashboard.
- Palette: warm white, soft graphite, a vivid but restrained blue accent, muted secondary text, and semantic status colors. Support light, dark, and follow-system themes.
- Smooth 180–280 ms transitions, gentle spring movement, meaningful hover/pressed/focus states, reduced-motion support, and polished loading, offline, error, denied, and success states.
- Shared design system across Windows, Android, and iOS, but preserve each mobile platform's native biometric dialogs, notification behavior, navigation conventions, and permission prompts.
- Accessibility: keyboard navigation on desktop, visible focus, screen-reader labels, high contrast, large text, and localization-ready strings.
- Do not imitate Apple's logo, lock-screen UI so closely that users are confused, or claim FaceKey is an Apple product.

## Main Windows desktop app screens

1. **Welcome / Setup** — FaceKey wordmark, a minimal security illustration, short value proposition, “Set up FaceKey”, and “How it works”.
2. **Home / Security status** — “Laptop protected” status, current unlock methods, paired phones, last activity, connection status, and a “Lock now” action only if implemented using supported OS capabilities.
3. **Unlock methods** — laptop-side Windows Hello face recognition only where a compatible IR camera exists; Windows Hello fingerprint only where a compatible fingerprint reader exists; paired-phone approval using fingerprint on supported Android phones or Touch ID on Touch ID iPhones; Windows PIN/password fallback. Never use phone face recognition for FaceKey approval. Dynamically detect real capabilities and label unavailable methods honestly.
4. **Pair a phone** — QR code and short pairing code, device name, verification on both devices, expiring one-time pairing challenge, cancel and revoke actions. Never use a permanent static QR as a credential.
5. **Unlock request** — Elegant, lock-inspired approval waiting state with laptop name, timestamp, abstract biometric icon, and accurate status such as “Waiting for approval on your phone”. This must not impersonate or replace the real Windows secure sign-in screen.
6. **Activity history** — Recent requests, approvals, denials, device names, timestamps and methods. Allow the user to clear local history.
7. **Settings** — Paired devices, optional nearby lock, grace period, notification preferences, theme, privacy controls, diagnostics, and device revocation.
8. **Help & recovery** — Test connection, check sensor compatibility, revoke a lost phone, and instructions for Windows Hello PIN/password recovery.

## Mobile companion app — Android AND iOS

Create a mobile app available for both platforms. Prefer a shared UI/codebase with Flutter if it speeds delivery, but use the official native authentication and secure-storage APIs through maintained plugins or carefully implemented platform channels. Do not simulate native biometrics.

### Shared phone features
- First-run setup and simple, secure onboarding.
- Pair the phone with a named Windows laptop by scanning the QR code or entering a short-lived pairing code.
- Recent approvals and denials, paired-device list, notification settings, help, lost-phone revocation, and privacy explanation.
- Recent approvals and denials, paired-device list, notification settings, help, lost-phone revocation, and privacy explanation.
- Actionable notifications may open a request screen, but must never silently approve a request. Require the user to unlock/authenticate and approve explicitly.
- Show precise permission and biometric capability states. If the device has no supported biometric enrolled, offer a documented system device-credential fallback only where safe and supported; otherwise decline phone approval and retain Windows fallback.

### Android-specific requirements
- Use AndroidX Biometric / BiometricPrompt configured for a fingerprint authenticator meeting the app's required strong biometric security policy; do not allow face recognition to approve a FaceKey unlock.
- Use Android Keystore for device-bound private keys; require user authentication for key use where supported.
- Handle Android version differences, manufacturer variations, unavailable or unenrolled fingerprint hardware, fingerprint lockout, device credential fallback policy, app backgrounding, and notification permission denial.
- Query authenticator capability at runtime. If a device exposes only face unlock or a fingerprint method that does not meet the security policy, mark fingerprint approval unavailable rather than quietly accepting a weaker method.
- Request only the Android permissions actually needed. Avoid continuous location collection. If Bluetooth or nearby-device access is used, explain it and make it optional.
- Follow Android app signing, secure storage, and release-build practices. No hard-coded secrets.

### iPhone-specific requirements
- Use Apple's LocalAuthentication API for **Touch ID only** on iPhone models that support it; Face ID must not be used for FaceKey phone approval.
- On an iPhone without Touch ID, offer an explicit system device-passcode fallback if enabled and suitable; otherwise report that fingerprint approval is unavailable on that device.
- Store device keys in Keychain and use Secure Enclave-backed keys where available and suitable.
- Do not configure or invoke Face ID for FaceKey approval. Handle user cancellation, lockout, missing Touch ID, and allowed passcode fallback.
- Never access or store biometric templates.

## Secure approval protocol

- Laptop generates a fresh, random, single-use, short-lived challenge for each request.
- Phone verifies the paired laptop identity and challenge context before prompting the user.
- After explicit approval and successful native local authentication, the phone signs that exact challenge with its device-bound private key.
- Laptop validates the signature, expiry, device pairing status, challenge uniqueness, and replay protection before accepting it.
- Use TLS, authenticated pairing, appropriate rate limits, key rotation and revocation, secure IPC, encrypted secret storage, and privacy-preserving logs.
- Reject expired, duplicated, malformed, revoked, or mismatched challenges. Never trust notification text alone as authorization.
- No biometric data, passcodes, or reusable private keys may be sent to the laptop or a server. Minimize telemetry and document data flows and retention.

## Windows security and OS limitations

- Target Windows 11 first, with a real desktop application using an appropriate native stack such as WinUI 3/.NET.
- Use Windows Hello / Windows Biometric Framework for supported in-app authentication and investigate supported Windows sign-in integration for genuine pre-login unlock.
- An ordinary desktop app cannot simply dismiss the Windows lock screen. True pre-login authentication is a separate OS-integrated Windows credential-provider/sign-in project and must be implemented only using supported mechanisms, with a threat model and recovery tests.
- If true pre-login integration is not safely completed, ship the desktop + phone pairing/approval experience as a clearly labelled **approval prototype**, and do not claim that tapping Approve actually unlocks the real Windows sign-in screen.
- Never weaken, bypass, or disable Windows security. Keep Windows Hello PIN/password recovery usable at all times.
- Proximity detection may optionally lock the laptop after a grace period, but proximity alone must never unlock it.

## Recommended implementation milestones

1. Inspect the existing repository, preserve working features, and build the polished Windows desktop shell with clearly marked demo states.
2. Build the shared Android/iOS companion UI and platform-specific capability detection.
3. Implement secure QR pairing with expiring challenge and two-sided confirmation.
4. Implement fingerprint-only phone approval and device-bound challenge signing: Android fingerprint on compatible Android phones; iPhone Touch ID on Touch ID models, with an explicit device-passcode fallback for Face ID-only iPhones if supported. Do not use phone face recognition.
5. Implement secure desktop verification, revocation and replay protection.
6. Add real Windows Hello capability detection and app authentication.
7. Investigate actual Windows pre-login integration separately; do not mark it complete until it works through supported OS mechanisms and is recovery-tested.
8. Test expired/replayed requests, revoked or lost phones, offline states, denied notifications, biometric failure/lockout, app restarts, clock changes, pairing cancellation, multiple paired phones, and recovery.

## Deliverables

- Working repository with readable, modular code and setup instructions.
- Windows desktop application, Android build, and iOS project/build instructions.
- README with platform prerequisites, supported hardware/OS table, build/run steps, and a clear feature-status matrix.
- Architecture diagram, threat model, data-flow description, pairing and revocation documentation.
- Unit/integration tests for pairing, challenge signatures, expiry, revocation, and replay attempts.
- Clearly isolated demo mode for UI previews/tests only. Never fake a successful biometric result in production.

## Acceptance criteria

- Consistent, premium iOS-inspired experience across Windows, Android and iPhone, with appropriate native interactions on each platform.
- Every visible control works; no fake security dialogs or dead buttons.
- Actual biometric capabilities are detected at runtime; laptop face recognition is laptop-only, laptop fingerprint uses Windows Hello where supported, and phone approval is fingerprint-only (Android fingerprint or iPhone Touch ID, with the documented passcode fallback where enabled).
- Biometric templates and secrets never leave their device.
- Every phone approval is explicit, bound to a fresh single-use challenge, expires quickly, and cannot be replayed.
- A lost or revoked phone cannot approve new requests.
- Windows fallback sign-in is always documented and available.
- Product documentation distinguishes a working pairing/approval prototype from genuine OS-level laptop unlocking.

Build an end-to-end vertical slice first instead of many nonfunctional screens. Before changing existing code, inspect the repository and preserve working features. Explain unsupported requirements rather than faking them.