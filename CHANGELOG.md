# SigChain Guard — Changelog

> **"Hardware doesn't lie."**
> Every validation is signed by the device itself. Not the app. Not the OS. The chip.

---

## v0.1.2 — June 15, 2026

The biggest internal overhaul since the first beta build. This release hardens the security architecture significantly, closes several attack surfaces surfaced during internal security testing, and lays the groundwork for production launch.

### Security

**API key moved to native memory on init**
On initialization the API key is immediately passed into the native library and stored there. After that point all security operations read it from native memory rather than managed C#. The key remains in the config asset for the Unity build pipeline but is not held in managed memory during runtime security operations.

**SDK version locked into native binary**
The SDK version is now a compile-time constant baked into `libscg.so` at build time. Any payload reaching the backend always carries the real version from the compiled library — the C# layer no longer has a version string to tamper with. Old SDK versions fail Gate 5 automatically.

**Expanded hardware scanner**
The runtime scanner received a major upgrade informed by dedicated threat research on Meta Quest hardware specifically. New detection capabilities include:

- Hook framework detection via thread name scanning, file descriptor analysis, and memory map inspection — catches injection tools even when they run without a network listener
- Inline hook detection via disk-to-memory comparison of key native libraries — any function patched in memory is detected regardless of how it was patched
- KernelSU detection via a userspace syscall behavior probe — a standard system call that returns an abnormal response when KernelSU is present, surviving filesystem-level concealment tools that hide root artifacts but cannot intercept the kernel's own manager interface
- APatch and advanced root framework artifact detection
- Memory editor process detection including no-root container execution modes
- Virtual container detection for tools that run your game inside a cloned environment
- Debugger attachment detection via multiple independent methods
- Environment variable injection detection

All scanner signals are bundled into the validation payload and evaluated server-side. The scanner does not make local ban decisions — it reports, the server decides.

**Removed `.so` from AAR packaging**
The native library is now distributed as a standalone plugin rather than bundled inside the AAR. This separates the Java bridge from the native binary, simplifying updates and eliminating a Gradle packaging conflict.

**HWID consistency fix**
The hardware device ID is now cached after the first full validation. All subsequent HWID reads return the value derived from the real hardware data — no more secondary HWID being generated from incomplete inputs.

### Architecture

**Java bridge replaced**
The old multi-file Java bridge (8 files, ~1200 lines) has been replaced with a single focused shim. Java now handles only what requires a hardware context — MediaDrm, KeyStore/TEE, PackageManager, and HTTPS. All cryptography, payload construction, scanning, and result handling moved into the native library.

**C# calls native directly**
The Unity integration layer now calls `libscg.so` directly via P/Invoke for all security operations. Java is called only for hardware APIs that cannot be accessed from native code. This removes an entire bridge layer from the attack surface.

**Initialisation state in native memory**
The SDK initialisation flag no longer exists as a C# boolean. It is tracked inside the native library, invisible to the managed runtime.

### Runtime Tamper Shield (SCG-RTS)

Full implementation of the runtime value protection system:

- Integer, float, and boolean value types supported
- Dual encrypted shadow copy storage with independent keys
- Automatic key rotation every 50 write operations
- Per-value bounds enforcement and write delta limiting
- Tamper detection fires immediately on shadow copy mismatch
- All violations are queued and bundled into the next validation payload — server receives a full picture of what was attempted

### Integration

**Single integration script**
`SCGAuth.cs` — attach to any GameObject in the first scene, configure three options in the Inspector. No code required for standard integration:
- Photon auth toggle (automatic JWT passthrough on success)
- Offline allow toggle
- Fail scene name

**Unity editor window — 4 tabs**
Rebuilt from scratch:
- **Setup** — API key management, BTKV injection, 9 project checks with one-click Fix All including automatic plugin import settings
- **Validate** — all SDK methods and result codes documented inline
- **RTS** — usage examples, supported types, protection details
- **Info** — SDK version read from the native binary, pre-launch checklist, links

**Plugin import auto-fix**
Fix All now correctly configures `libscg.so` import settings — platform, architecture, and load order — eliminating a class of build failures that required manual Inspector configuration.

---

## v0.0.1-beta — May 2026

Initial private beta release.

- 8-gate server-side validation pipeline
- Hardware TEE attestation against Meta hardware root CA
- APK certificate baseline locking
- Permanent HWID derivation from factory-burned device values
- AES-256-GCM encrypted, HMAC-signed payload
- Nonce replay prevention
- Cross-game ban database
- Photon custom auth integration
- Unity 2022.3 LTS and Unity 6 support
- Meta Quest 3 and Quest 3S validated

---

*SigChain Guard is currently in private beta. Production launch coming soon.*
*support@sigchainguard.com*
