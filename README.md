# SigChain Guard

**Hardware-level anti-cheat and app integrity verification for Unity VR games.**

> ⚠️ **NOT RELEASED — DO NOT CREATE AN ACCOUNT**
> SigChain Guard is currently in active development and testing. The dashboard and backend are not production-ready. Any accounts created right now will be deleted. Please wait until this notice is removed before signing up. This README is updated every time there is a significant change or milestone.

---

## 🧪 Beta Tester Program — 10 Spots Available

We are looking for **10 developers** to join our closed beta testing program.

**What you get:**
- 1 game of your choice protected by SCG **free for life**
- Early access before public launch
- Direct input into how the product is shaped

**What we ask:**
- Test SCG with a real or test game on Meta Quest
- Share your experience, bugs, and feedback via [support@sigchainguard.com](mailto:support@sigchainguard.com)
- Be honest — good or bad, we want to know

If you are interested, email [support@sigchainguard.com](mailto:support@sigchainguard.com) with the subject line **"Beta Tester"**. Spots are first come first served and limited to 10.

---

## What is SigChain Guard?

SigChain Guard (SCG) is an anti-cheat SDK built specifically for VR games on Meta Quest and Android VR headsets. Unlike traditional anti-cheat systems that rely on software checks alone, SCG roots its trust in the device's hardware — specifically the Trusted Execution Environment (TEE) built into the chip itself.

Every validation is cryptographically signed by the device hardware. The server never trusts anything the client says — it verifies everything independently. If the APK has been tampered with, the device is rooted, or hooks like Frida or Xposed are present, SCG catches it.

---

## Why SigChain Guard?

Most anti-cheat solutions are built for PC or mobile. VR is different — sideloading is common, devices are frequently developer-unlocked, and the threat model is unique. SCG was designed from the ground up for this environment.

**What makes it different:**

- **Hardware TEE attestation** — The chip signs every validation. You cannot fake this without the physical device and its private keys
- **APK certificate binding** — Your signing certificate is registered at setup. Any re-signed or tampered build is caught instantly
- **Permanent hardware banning** — HWIDs are derived from factory-burned device values. Bans survive reinstalls, account changes, and factory resets
- **Cross-game ban database** — One ban affects every SCG-protected game
- **Continuous re-validation** — Passing launch is not enough. SCG re-validates silently mid-session
- **Zero client trust** — The server makes every decision. The client collects and sends — nothing more
- **8 validation gates** — Payload integrity, SDK version control, timestamp, nonce replay prevention, TEE cert chain, APK baseline, developer-configured checks
- **Built for VR** — Works with sideloaded apps, dev builds, Meta Store, AppLab. No dependency on Google Play Services
- **2FA on all accounts** — Email-based two-factor authentication on every login
- **Session security** — Sessions are cryptographically bound to your device — stolen cookies are useless on another machine

---

## Who Made This?

I built SigChain Guard. I won't specify my age for privacy reasons.

To be fully transparent — I did not write any of this code myself. I used AI as a coding assistant to implement everything. Every architecture decision, security gate, verification flow, and design choice came from me — I directed all of it — but the actual code was written by AI based on my instructions and logic.

I am not a security expert or a professional developer. I am someone actively trying to learn cybersecurity and development, figuring things out as I go, and doing my best to build something genuinely useful and secure. I do not claim to be a genius of any kind. I just try hard, research thoroughly, and test everything on real hardware (Meta Quest 3S).

If there are bugs or security issues — and there may be, this is an early beta — I take them seriously. Report anything at [support@sigchainguard.com](mailto:support@sigchainguard.com) and I will respond and do my best to fix it.

---

## Current Status

**Early beta — testing and bug fixing in progress.**

**What is working:**
- Full 8-gate validation pipeline — tested and passing on Quest 3S
- SDK version control (Gate 5) — deprecated versions blocked instantly
- BTKV game registration flow — APK upload, cert extraction, token generation, verification
- Auth system — signup, email verification, signin, password reset, 2FA with email OTP
- Session fingerprinting — sessions bound to device IP + user agent
- Dashboard settings — view API key (password protected), delete game (OTP verified), 2FA management
- Hardware HWID generation and tracking
- TEE attestation — verified against device hardware root CA
- APK baseline fingerprinting — cert + package locked after 3 confirmations
- Unity SDK — SCG.Validate(), ReValidate(), ValidatePS(), HWIDPlayer(), ReturnHwid()
- Unity setup window — settings checker, Fix All, BTKV inject/remove, API key management
- Proguard/R8 obfuscation with seed-based package repackaging

**What is still being worked on:**
- Billing integration (Lemon Squeezy)
- Dashboard real data (validations, players, MAU stats)
- Unity package finalization and clean project testing
- Photon integration in Unity editor window
- JWT server-side verification endpoint
- Docs page
- Final security audit before launch

---

## Get Started (When Released)

**Website & Dashboard:** [sigchainguard.com](https://sigchainguard.com)

Once released: create an account, register your game via the BTKV verification flow, get your API key, and drop the Unity package into your project. Integration takes under 30 minutes.

---

*SigChain Guard — hardware-level VR anti-cheat. Built by a developer trying to learn, for developers who want real security.*

My goal is simple: make good security affordable. VR anti-cheat shouldn't only be available to big studios with big budgets. If you have knowledge, experience, or ideas that could help make SCG better — I genuinely welcome it. Reach out at [support@sigchainguard.com](mailto:support@sigchainguard.com).
