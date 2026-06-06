# SigChain Guard

**Hardware-level anti-cheat and app integrity verification for Unity VR games.**

> ⚠️ **NOT RELEASED — DO NOT CREATE AN ACCOUNT**
> SigChain Guard is currently in active development and testing. The dashboard and backend are not production-ready. Any accounts created right now will be deleted. Please wait until this notice is removed before signing up. This README is updated every time there is a significant change or milestone.

---

## 🧪 Beta Tester Program — 7 Spots Remaining

We are looking for developers to join our closed beta testing program. **3 spots have already been filled.**

**What you get:**
- 1 game of your choice protected by SCG **free for life**
- Early access before public launch
- Direct input into how the product is shaped

**What we ask:**
- Test SCG with a real or test game on Meta Quest
- Share your experience, bugs, and feedback via [support@sigchainguard.com](mailto:support@sigchainguard.com)
- Be honest — good or bad, we want to know

If you are interested, email [support@sigchainguard.com](mailto:support@sigchainguard.com) with the subject line **"Beta Tester"**. Spots are first come first served.

---

## What is SigChain Guard?

SigChain Guard (SCG) is an anti-cheat SDK built specifically for VR games on Meta Quest and Android VR headsets. Unlike traditional anti-cheat systems that rely on software checks alone, SCG roots its trust in the device's hardware — specifically the Trusted Execution Environment (TEE) built into the chip itself.

Every validation is cryptographically signed by the device hardware. The server never trusts anything the client says — it verifies everything independently. If the APK has been tampered with, the device is rooted, or known hook frameworks are present, SCG catches it.

---

## Why SigChain Guard?

Most anti-cheat solutions are built for PC or mobile. VR is different — sideloading is common, devices are frequently developer-unlocked, and the threat model is unique. SCG was designed from the ground up for this environment.

**What makes it different:**

- **Hardware TEE attestation** — The chip signs every validation. You cannot fake this without the physical device and its private keys
- **APK certificate binding** — Your signing certificate is registered at setup. Any re-signed or tampered build is caught instantly
- **Permanent hardware banning** — HWIDs are derived from factory-burned device values. Bans survive reinstalls, account changes, and factory resets
- **Cross-game ban network** — A ban in one SCG-protected game is visible to every other SCG-protected game on the network
- **Continuous re-validation** — Passing at launch is not enough. SCG re-validates silently mid-session
- **Zero client trust** — The server makes every decision. The client collects and sends — nothing more
- **8 validation gates** — Layered server-side checks that cannot be individually bypassed
- **Built for VR** — Works with sideloaded apps, dev builds, Meta Store, and AppLab. No dependency on Google Play Services
- **Photon integration** — Native support for Photon custom auth. Banned or failed players are blocked from joining rooms entirely
- **2FA on all accounts** — Email-based two-factor authentication on every login
- **Session security** — Sessions are cryptographically bound to your device

---

## What Developers Are Saying

> *"Works really well out of the box. If you know basic C# the docs walk you through everything clearly. The dashboard is a standout — being able to toggle custom auth on and off without touching code is genuinely useful. Would recommend it to anyone building a VR multiplayer game."*
>
> — **Bonk**, GTag Locomotion Developer · Beta Partner

---

## Who Made This?

I built SigChain Guard. I won't specify my age for privacy reasons.

To be fully transparent — I did not write any of this code myself. I used AI as a coding assistant to implement everything. Every architecture decision, security model, verification flow, and design choice came from me — I directed all of it — but the actual code was written by AI based on my instructions and logic.

I am not a security expert or a professional developer. I am someone actively learning cybersecurity and development, figuring things out as I go, and doing my best to build something genuinely useful and secure. I do not claim to be anything I am not. I just try hard, research thoroughly, and test everything on real hardware (Meta Quest 3S).

If there are bugs or security issues — and there may be, this is an early beta — I take them seriously. Report anything at [support@sigchainguard.com](mailto:support@sigchainguard.com) and I will respond promptly.

---

## Current Status

**Early beta — testing and refinement in progress.**

**What is working:**
- Full 8-gate validation pipeline — tested and passing on Quest 3S
- SDK version control — deprecated versions blocked instantly
- BTKV game registration flow — APK upload, cert extraction, token generation, verification
- Auth system — signup, email verification, signin, password reset, 2FA with email OTP
- Session fingerprinting — sessions cryptographically bound to device
- Dashboard — API key management, game settings, player panel, ban/unban, network settings
- Hardware HWID generation and tracking
- TEE attestation — verified against device hardware root CA
- APK baseline fingerprinting — cert and package locked after confirmed registrations
- Unity SDK — Validate(), ReValidate(), ValidatePS(), HWIDPlayer(), ReturnHwid(), GetLastToken()
- Unity setup window — settings checker, Fix All, BTKV flow, API key management
- Photon custom auth integration — banned players blocked at room join level
- Cross-game ban network — network-wide bans and auto-ban thresholds
- Per-game ban messages — custom ban text configurable from dashboard
- MAU tracking and quota enforcement
- Obfuscation — ProGuard/R8 with seed-based package repackaging

**What is still being worked on:**
- Billing integration
- Dashboard analytics (validations, players, MAU graphs)
- Unity package final cleanup and clean project testing
- Docs page
- Final security review before public launch

---

## Planned for the Future

- **Native library migration** — Moving core security logic from the Java layer into a compiled native library for significantly stronger resistance to reverse engineering and runtime hooking
- **Runtime Shield (SCG-RTS)** — An optional layer that protects critical game values like health, score, and currency from memory editors. Values are stored encrypted in native memory with tamper detection that reports violations to the server
- **Lemon Squeezy billing** — Subscription management directly from the dashboard
- **Player ID system** — Short anonymous player identifiers for ban management and support without exposing hardware IDs
- **Broadcast system** — Dashboard-level announcements pushed to all active SDK integrations
- **Multi-region infrastructure** — Redundancy across multiple hosting regions for improved uptime and latency

---

## Get Started (When Released)

**Website & Dashboard:** [sigchainguard.com](https://sigchainguard.com)

Once released: create an account, register your game via the BTKV verification flow, get your API key, and drop the Unity package into your project. Integration takes under 30 minutes.

---

*SigChain Guard — hardware-level VR anti-cheat. Built by a developer trying to learn, for developers who want real security.*

My goal is simple: make good security accessible. VR anti-cheat shouldn't only be available to big studios with big budgets. If you have knowledge, experience, or ideas that could help make SCG better — I genuinely welcome it. Reach out at [support@sigchainguard.com](mailto:support@sigchainguard.com).
