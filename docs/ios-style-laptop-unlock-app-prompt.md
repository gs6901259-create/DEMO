# Product Prompt: FaceKey — iOS-Inspired Laptop Unlock

Build a polished Windows 11 desktop application and iPhone companion app called **FaceKey** (working name). The product should feel like a premium Apple/iOS experience: quiet, elegant, minimal, glass-like surfaces used sparingly, system-quality typography, subtle motion, rounded corners, coherent icons, and excellent accessibility. Do not copy Apple's logo or pretend this is an official Apple product.

## Product goal
Make unlocking a Windows laptop feel effortless and premium. Support Windows Hello face/fingerprint sign-in where compatible, plus an optional phone-based approval flow in which the user approves a pending unlock request on their paired phone using Face ID or Touch ID (or the phone's native biometric/passcode fallback). The phone must never transmit fingerprint/face data to the laptop; biometric verification happens locally on the phone, and the app receives only a cryptographic approval result.

## Design direction
- Overall visual system inspired by iOS: SF-like system typography (use the platform system font, with a high-quality cross-platform fallback), generous whitespace, restrained color, carefully layered translucent materials, soft shadows, thin dividers, and 14–24 px corner radii.
- Palette: warm white / soft graphite, Apple-like blue accent, muted secondary text, semantic green/orange/red; a full dark mode that respects Windows/iOS system appearance.
- Smooth 180–280 ms transitions, spring-like panel movement, subtle hover/press states, reduced-motion support, no excessive neon/glow, no crowded dashboard, no fake 3D face scanner.
- Responsive desktop window and phone-first companion screens. Beautiful first-run onboarding, clear empty/loading/success/failure states, polished tray/menu-bar-like Windows notification-area presence, and useful settings.
- Accessibility: keyboard navigation, visible focus indicators, high contrast option, screen-reader labels, large text support, localization-ready strings.

## Main desktop app screens
1. **Welcome / Setup** — FaceKey wordmark, simple illustration/icon, one sentence explaining secure laptop access, “Set up FaceKey” primary action, “How it works” secondary action.
2. **Home / Security status** — large calm status card showing “Laptop protected”, current method, paired phone, last unlock, connection state, and a clear “Lock now” action (only if implemented through supported OS APIs).
3. **Unlock methods** — Windows Hello face recognition, Windows Hello fingerprint, paired iPhone approval, and Windows PIN/password fallback. Display capability status based on real hardware and configuration; never claim a camera or sensor is supported unless verified.
4. **Phone pairing** — QR code plus short pairing code, explicit device name, verification on both devices, expiry for pairing challenge, cancel/revoke actions, and clear instructions. Do not use a static QR as a permanent credential.
5. **Unlock request screen** — polished lock-inspired screen with a face-ring/fingerprint-inspired abstract icon, laptop name, request timestamp, and status text such as “Waiting for your iPhone approval”. The app UI must not replace the genuine Windows sign-in/secure desktop with a simulated login.
6. **Activity** — locally stored recent unlock requests and approvals with time, method, and device; allow clear-history action.
7. **Settings** — paired devices, auto-lock/nearby presence preferences, proximity threshold, notification preferences, theme (light/dark/system), privacy controls, security diagnostics, and remove-this-device flow.
8. **Help & recovery** — test connection, troubleshoot camera/biometric compatibility, revoke paired phone, and always show a reliable Windows sign-in fallback.

## iPhone companion screens
- iOS-native-feeling welcome/onboarding.
- Securely pair this iPhone to a named laptop.
- Approval screen showing laptop identity, approximate request context, and Approve / Deny actions.
- Require LocalAuthentication (Face ID / Touch ID / device passcode fallback) immediately before approval. Never collect/store Face ID or Touch ID templates.
- Device list, revoke device, notification preferences, recent approvals, and privacy/security explanation.
- Push notifications should be actionable but never approve silently. Require user presence and a fresh cryptographic challenge for every approval.

## Core functionality and architecture
- Target **Windows 11 first**. Build a real desktop app, not merely a website that pretends to unlock Windows. Choose a suitable native Windows approach (WinUI 3/.NET or another well-supported desktop stack) and a native iOS companion (SwiftUI). If using a shared UI framework, preserve native OS authentication APIs.
- Use Windows Hello / Windows Biometric Framework or supported Windows sign-in integration for native biometric sign-in. The app's own biometric prompt must not be represented as a replacement for Windows sign-in.
- Important OS limitation: an ordinary user-mode app cannot simply dismiss the Windows lock screen by toggling a local flag. Treat actual pre-login unlocking as a separate OS-integrated component and implement only supported Windows credential-provider / sign-in integration, with documented setup and recovery. If this cannot be completed safely in the first release, build the companion authentication demo and dashboard honestly, but label it “approval prototype” and do not claim it unlocks Windows yet.
- Phone approval protocol: laptop generates a fresh, random, short-lived challenge; phone verifies laptop identity and challenge; phone requires biometric/passcode confirmation; phone signs the challenge with a device-bound private key protected by the platform secure hardware/keychain where available; laptop validates the signature, expiry, device pairing, nonce uniqueness, and replay protection. Use TLS, authenticated pairing, key rotation/revocation, rate limiting, and audit logs that avoid storing biometric data.
- Store no raw face images or fingerprints. Do not upload biometric data. Minimize collected data, encrypt secrets at rest, secure IPC, and document threat model and permissions.
- Require explicit opt-in for proximity-based locking/unlocking. Proximity alone must never silently unlock the computer. Lock on proximity loss can be an optional convenience, with a grace period and recovery controls.
- Support offline recovery through Windows Hello PIN/password and revoking a lost phone. Never lock users out if the phone is unavailable.
- Prevent replay, spoofed notifications, unauthorized remote commands, and insecure hard-coded secrets. No default credentials or embedded production API keys.

## Recommended implementation milestones
1. Build the visual shell and responsive screen states with realistic demo data clearly marked as demo.
2. Implement desktop-to-iPhone pairing with QR + expiring challenge.
3. Implement challenge signing and biometric-gated approval on iPhone.
4. Add genuine Windows Hello capability detection and app authentication.
5. Investigate supported Windows sign-in integration for true pre-login unlock; ship only when security/recovery testing is complete.
6. Add tests for expired challenges, repeated/replayed approvals, revoked devices, offline phone, notification denial, failed biometrics, app restarts, and account recovery.

## Deliverables
- Working repository with clean architecture and readable code.
- README with setup/run instructions for Windows and iOS, required tooling, hardware requirements, and a feature-availability table.
- Architecture diagram, threat model, data-flow description, and pairing/unpairing flow.
- Tests for authentication and pairing logic.
- Never fake successful biometric authentication in production. Provide a clearly isolated demo mode only for UI previews and automated tests.

## Acceptance criteria
- The app looks and feels like a premium, consistent iOS-inspired product on both devices.
- All visible controls do real things; no dead buttons or fake system dialogs.
- UI accurately shows which unlock methods are actually available on the user's hardware.
- Face/fingerprint data never leaves the device performing biometric verification.
- Phone approvals are bound to a fresh challenge, expire quickly, and cannot be replayed.
- Windows fallback sign-in remains available at all times.
- Documentation clearly distinguishes a working UI prototype from real OS-level laptop unlocking.

Build this incrementally. Prioritize a polished working vertical slice over a huge set of nonfunctional screens. Before changing an existing codebase, inspect its structure and preserve working features; explain any unsupported requirement instead of simulating it.