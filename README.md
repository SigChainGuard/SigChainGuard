# SigChain Guard

**Hardware-level anti-cheat and app integrity verification for Unity VR games.**

> ⚠️ **NOT RELEASED — DO NOT CREATE AN ACCOUNT**
> SigChain Guard is currently in active development and testing. The dashboard and backend are not production-ready. Any accounts created right now will be deleted. Please wait until this notice is removed before signing up. This README is updated every time there is a significant change or milestone.

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
- **8 validation gates** — Payload integrity, SDK version, timestamp, nonce replay, bridge hash, TEE cert chain, APK baseline, developer-configured checks
- **Built for VR** — Works with sideloaded apps, dev builds, Meta Store, AppLab. No dependency on Google Play Services

---

## Who Made This?

I built SigChain Guard. I won't specify my age for privacy reasons.

To be fully transparent — I did not write any of this code myself. I used AI as a coding assistant to implement everything. Every architecture decision, security gate, verification flow, and design choice came from me — I directed all of it — but the actual code was written by AI based on my instructions and logic.

I am not a security expert or a professional developer. I am someone actively trying to learn cybersecurity and development, figuring things out as I go, and doing my best to build something genuinely useful and secure. I do not claim to be a genius of any kind. I just try hard, research thoroughly, and test everything on real hardware (Meta Quest 3S).

If there are bugs or security issues — and there may be, this is an early beta — I take them seriously. Report anything at [support@sigchainguard.com](mailto:support@sigchainguard.com) and I will respond and do my best to fix it.

---

## Current Status

**Early beta — testing and bug fixing in progress.**

What is working:
- Full 8-gate validation pipeline on Quest 3S
- BTKV game registration flow
- Auth system (signup, email verification, signin, password reset)
- Dashboard UI (basic)
- Hardware HWID generation and tracking
- TEE attestation with Meta root CA

What is still being worked on:
- Bridge hash system (per-SDK version)
- Billing integration
- Dashboard real data (validations, players, stats)
- Unity package finalization
- Docs
- Security audit

---

## Get Started (When Released)

**Website & Dashboard:** [sigchainguard.com](https://sigchainguard.com)

Once released: create an account, register your game via the BTKV verification flow, get your API key, and drop the Unity package into your project.

---

*SigChain Guard — hardware-level VR anti-cheat. Built by a developer trying to learn, for developers who want real security.*

My goal is simple: make good security affordable. VR anti-cheat shouldn't only be available to big studios with big budgets. If you have knowledge, experience, or ideas that could help make SCG better — I genuinely welcome it. Reach out at [support@sigchainguard.com](mailto:support@sigchainguard.com).
