# SigChain Guard — Security Overview

> **"Hardware doesn't lie."**
>
> This document explains how SigChain Guard (SCG) works, what it actually protects against,
> and — just as importantly — what it does **not** claim to do. It is written for two readers:
> the developer evaluating whether to trust SCG with their game, and the player who wants to
> know what runs on their headset and what data leaves it.
>
> We would rather lose a sale to honesty than win one with a claim we can't defend.
> If anything here ever stops being true, this file changes. It is versioned alongside the
> [CHANGELOG](./CHANGELOG.md).

**Status:** Private beta · SDK not yet publicly released · `v0.1.2`

---

## Table of contents

1. [The one-sentence version](#the-one-sentence-version)
2. [Why secrecy is not our security model](#why-secrecy-is-not-our-security-model)
3. [How a validation actually works](#how-a-validation-actually-works)
4. [The trust anchor: hardware attestation](#the-trust-anchor-hardware-attestation)
5. [The 8 gates](#the-8-gates)
6. [What an attacker can and cannot do (threat model)](#threat-model)
7. [Claims we make — and the proof behind each](#claims-and-proof)
8. [Honest limitations](#honest-limitations)
9. [Player data and privacy](#player-data-and-privacy)
10. [Device identity, bans, and the appeal path](#device-identity-bans-and-appeals)
11. [What we deliberately do not document](#what-we-deliberately-do-not-document)

---

## The one-sentence version

SCG does not ask the app *"are you legitimate?"* — it asks the **device's secure hardware** to
cryptographically prove the app and the device are genuine, and then verifies that proof on our
server against the hardware manufacturer's root certificate. The app is never trusted. The
hardware signature is.

---

## Why secrecy is not our security model

A common anti-cheat design hides its logic and hopes nobody reverse-engineers it. That is
**security through obscurity**, and it fails the moment someone opens the APK.

SCG is built on the opposite principle, the one modern cryptography is founded on
([Kerckhoffs's principle](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle)): **a system
should remain secure even if everything about how it works is public — as long as the keys stay
secret.** For SCG, the "key" is not in the app at all. It is a private key fused into the
device's Trusted Execution Environment (TEE) at the factory, which no software on the device —
not even the operating system — can read or extract.

This is why we can publish this document in full detail. Knowing *how* SCG works does not help
you defeat it, because defeating it requires forging a hardware signature you cannot produce
without the physical secure element's cooperation.

---

## How a validation actually works

A single `SCG.Validate()` call runs this sequence. Nothing here is secret; the security is in
step 6, not in hiding the steps.

```
 1. App requests a one-time nonce (random challenge) from the SCG backend.
 2. The device's hardware collects signals:
       • a TEE-attested key + certificate chain (the core proof)
       • the running APK's signing certificate
       • device model / integrity signals
       • a runtime security scan result
 3. A hardware-derived device identifier is computed.
 4. The payload is encrypted (AES-256-GCM) and signed (HMAC).
 5. It is sent over HTTPS to the SCG backend.
 6. The backend INDEPENDENTLY verifies everything — see "The 8 gates" below.
       The backend trusts none of the client's claims; it re-derives and re-checks each one.
 7. On success, the backend issues a short-lived signed token (JWT).
 8. Your game receives PASSED / FAILED / a specific reason code.
```

The important property: **a `PASSED` result is minted by our server, not by the client.** A
modified app can return whatever it wants to its own code, but it cannot manufacture a server
token, because the token is only issued after the hardware proof checks out on our side.

---

## The trust anchor: hardware attestation

This is the heart of SCG, so it's worth being precise.

Modern Android devices — including Meta Quest headsets — ship with a **hardware-backed keystore**
inside a TEE. When an app asks the keystore to generate a key with an *attestation challenge*,
the TEE produces the key **inside the secure hardware** and returns an X.509 certificate chain
that:

- is signed by a key that **never leaves the secure element**, and
- chains up to a **root certificate held by the hardware manufacturer** (for Meta Quest, the
  Facebook/Meta hardware attestation root; on other devices, Google's hardware root), and
- embeds verifiable facts about the device state — including the app package name, the app's
  signing-certificate digest, whether the bootloader is locked, and the verified-boot state.

SCG's backend verifies that chain server-side. Concretely, the verifier checks (illustrative
shape, not the full implementation):

```text
verifyAttestation(chain, expectedNonce, expectedPackage, expectedCertDigest):
    1. Parse the certificate chain.
    2. Walk the chain and confirm each certificate is signed by the next.
    3. Confirm the ROOT matches the known hardware manufacturer root
         (pinned fingerprint — we compare against the real published root).
    4. Read the attestation extension from the leaf certificate:
         - challenge        must equal the nonce WE issued this session  → blocks replay
         - packageName      must equal the developer's registered app
         - certDigest       must equal the developer's registered signing cert
         - bootState/locked must indicate a genuine, locked, verified device
    5. Only if ALL of the above hold → this gate passes.
```

Why this matters in plain terms:

- The certificate **cannot be forged** without the device's secure-element private key, which is
  not extractable by software.
- It **cannot be replayed** from another session, because it must contain the exact random nonce
  our server generated moments earlier.
- It **cannot be lifted from another app or another device**, because the embedded package name,
  signing-cert digest, and device-bound key won't match.

A real, redacted example of what our backend logs when it verifies a genuine Quest 3S
(values truncated):

```
[SCG TEE] chain length: 4
[SCG TEE] Cert 3 subject: CN=Facebook Hardware Attestation Root CA
[SCG TEE] Root fingerprint matches pinned root: true
[SCG TEE] challenge matches issued nonce: true
[SCG TEE] packageName: com.developer.theirgame   (matches registered: true)
[SCG TEE] certFingerprint: A5:54:…:BD   (matches registered: true)
[SCG TEE] deviceLocked: true   verifiedBootState: VERIFIED
[SCG TEE] ALL CHECKS PASSED
```

---

## The 8 gates

Every validation must pass **all eight** server-side gates. Each is verified independently on
our backend; the client cannot skip or satisfy any of them by lying.

| # | Gate | What it confirms |
|---|------|------------------|
| 1 | **Required fields** | The payload is well-formed and complete. |
| 2 | **SDK version + bridge integrity** | The request comes from a current, unmodified SDK build. |
| 3 | **Timestamp** | The request is fresh, within a tight server-checked window — blocks stale/replayed payloads. |
| 4 | **Nonce** | The one-time challenge matches the one we issued — blocks replay. |
| 5 | **SDK not deprecated** | Old/withdrawn SDK versions are rejected. |
| 6 | **Hardware TEE attestation** | The core proof — see the section above. |
| 7 | **APK certificate baseline** | The app's signing certificate matches what the developer registered. |
| 8 | **Developer-configured checks** | Your dashboard rules (root policy, scan-flag thresholds, ban lists, etc.). |

A ban-status check runs immediately after Gate 1, so banned devices are rejected early.

---

## Threat model

Security claims are meaningless without stating **what attacker you're defending against.**
Here is ours, honestly.

### What SCG stops on a normal, non-rooted, locked headset

- **Editing or recompiling the APK.** A repackaged build presents a different signing
  certificate (fails Gate 7) and cannot produce a matching attestation.
- **Faking a `PASSED` to the network.** The pass token is minted server-side after hardware
  verification; a client cannot forge it.
- **Replaying a captured valid payload.** The nonce (Gate 4) and timestamp (Gate 3) make a
  captured payload useless in any later session.
- **Lifting another device's or another app's attestation.** The embedded nonce, package name,
  signing-cert digest, and device-bound key won't match.
- **Running under common injection/hook tooling, emulators, or cloned containers** — these are
  detected by the runtime scan and surfaced to the developer (Gate 8).

### What raises the difficulty sharply but is part of our active threat surface

- **A rooted device or an unlocked bootloader.** Root fundamentally changes what software on the
  device can do. We detect root, unlocked bootloader, and a range of root-hiding frameworks, and
  we report verified-boot state from the hardware attestation itself — so a developer can choose
  to refuse rooted devices outright. We are honest that root is an arms race; our position is to
  **detect and let the developer decide**, not to claim root is impossible to achieve.

### What SCG does **not** claim to defend against

- **A compromise of the device's secure element / TEE itself.** If an attacker defeats the
  hardware secure element on a specific device, the hardware trust anchor for *that device* is
  broken. We are not aware of a general, practical method to do this on current locked retail
  headsets — but we will not claim it is "impossible," because that is not a claim anyone can
  honestly make about future hardware attacks.
- **In-headset behavioral cheats that require no software modification** (e.g. a player's own
  skill aids that don't touch the binary or memory). SCG verifies *integrity and identity*, not
  gameplay fairness heuristics.
- **Anything after a factory reset re-establishes a "fresh" device** — see Limitations.

---

## Claims and proof

We have deliberately removed superlatives like "impossible," "unhackable," and "reveals
nothing" from our marketing, because security professionals correctly distrust them. Here is
what we *do* claim, each stated so it can be checked.

| Claim | Why it holds | How you can verify |
|-------|--------------|--------------------|
| A repackaged/re-signed APK is rejected. | The signing-cert digest is embedded in the hardware attestation and checked against the developer's registered cert (Gates 6 + 7). | Re-sign a test build and watch it fail validation. |
| A `PASSED` cannot be forged client-side. | The pass token is issued by our server only after attestation verification; the signing key is server-side. | Inspect the SDK — there is no client path that mints a valid server token. |
| A captured payload can't be replayed later. | Each request must carry the exact server-issued nonce and a fresh timestamp (Gates 3 + 4). | Capture a valid payload and resend it; it is rejected. |
| Attestation can't be borrowed from another device/app. | The leaf certificate is device-bound and embeds package name + cert digest + nonce. | Send a genuine attestation from device A for app A against app B; it fails. |
| We verify against the real hardware root. | The backend pins the manufacturer's published root-CA fingerprint and rejects chains that don't terminate there. | The root fingerprint we check is the manufacturer's published value. |

If you are a security researcher and can demonstrate any of the above claims failing on a
non-rooted retail device, we want to hear from you: **support@sigchainguard.com**.

---

## Honest limitations

Trust is built by stating these plainly.

- **Factory reset.** On a non-privileged app, we cannot guarantee a device is re-identified
  after a full factory reset. Some hardware signals can change or be regenerated. We treat device
  identity as **strong but not absolute** and are moving to a multi-signal model (see below).
- **OS updates / reprovisioning.** Manufacturer updates can occasionally change the value
  returned by some hardware identifiers. Our multi-signal approach is designed to tolerate one
  signal drifting without losing the device's identity.
- **Root is an arms race.** We detect today's common root and root-hiding tools. New techniques
  appear; we update. We do not claim permanent, total root immunity.
- **No anti-cheat is a silver bullet.** SCG raises the cost of cheating dramatically and gives
  developers hardware-grounded signals and control. It is one strong layer, not a guarantee of a
  cheat-free game.
- **We are early.** SCG is in private beta on a young platform. We expect edge cases, and we have
  built an appeal path precisely so legitimate players are never stuck.

---

## Player data and privacy

What leaves a player's headset, and what we store:

- **Collected:** hardware identity *signals* (hashed device fingerprints), the APK certificate
  data, the TEE attestation result, and the runtime security-scan result.
- **Not collected:** no name, email, account handle, location, contacts, or other personally
  identifying information is gathered by the SDK from the player.
- **Hashing:** device signals are hashed; we store derived identifiers, not raw hardware serials.
- **Server-side keying:** stored device identifiers are **salted/keyed server-side**, so a
  database leak does not yield a globally reversible map of "hash → device" that anyone could
  recompute. (This is a deliberate design choice against the "leaked fingerprint database"
  risk.)
- **In transit:** everything is encrypted (AES-256-GCM) and sent over HTTPS.

The authoritative, legally binding description lives in the
[Privacy Policy](https://sigchainguard.com/privacy/). If this section and the Privacy Policy ever
disagree, the Privacy Policy governs.

---

## Device identity, bans, and appeals

- **How identity works.** Each device maps to a hashed identifier derived from hardware-backed
  values that a normal app is permitted to read. We are moving toward a **multi-signal / quorum**
  model: several independent signals are stored separately, and a device is matched on a majority
  rather than a single value — so one signal changing (e.g. after an OS update) does not create a
  "new" device or wrongly break a match.
- **Bans are the developer's decision.** SCG never bans a player on its own. It reports signals;
  the developer configures what counts as a fail and whether it is a soft warning or a hard
  block. Cross-game banning is opt-in per developer.
- **Appeals.** Because a hardware-level block on a young system can affect a legitimate player
  (a buggy edge case, an unusual device, a reprovisioned headset), we provide a documented
  appeal path. A flagged player is never left without recourse:
  **support@sigchainguard.com**. Appeals are reviewed against the stored signals, and confirmed
  false positives are cleared.

---

## What we deliberately do not document

In keeping with "explain the design, don't hand out a bypass," this document intentionally omits
operational specifics that would only serve someone trying to evade detection — for example, the
exact heuristics, thresholds, and signal combinations used by the runtime scanner, and the
precise internal scan ordering. None of these omissions are load-bearing for the security
argument above: SCG's protection rests on hardware attestation that does not depend on those
details staying secret. We withhold them only so that the day-to-day cat-and-mouse against
active cheating tools isn't made trivially easy.

If you are a developer doing due diligence and need more depth than this document provides,
contact us directly and we can talk under NDA.

---

*Questions, security reports, or due-diligence requests: **support@sigchainguard.com***
*This document is versioned with the project and updated whenever the system changes.*
