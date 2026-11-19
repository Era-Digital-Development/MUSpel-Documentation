# MÚSpel Engine — Core Modules Reference

*Modular Unified System Program Execution Layer — Core Modules Reference*

*****CONFIDENTIAL*****

**Project:** MÚSpel Engine  
**Document Version:** 1.0  
**DID:** ED-MSPL-MODL-001  
**Date:** 02 June 2026  
**Author:** Eric Slangen  
**Proprietary to:** Era Digital  
**© 2025–2026 Era Digital. All rights reserved.**

**Status:** Current

---

## 1. Overview

This document is the authoritative reference for every module in the MÚSpel Engine. It defines each module's name, Norse origin, single responsibility, layer, and implementation language.

**The single responsibility principle is non-negotiable.** Every module does exactly one thing. If a module accumulates a second responsibility, that is a design failure — not a feature. When a module needs something outside its responsibility, it communicates via Hermóðr (point-to-point) or Gjallarhorn (broadcast). It never reaches into another module directly.

**Implementation language:** The engine is primarily C#/.NET 10 with Native AOT. Three modules are implemented in C++ for justified reasons — Vǫlundr (cryptographic core), Surtr (memory protection), and Níðhǫggr (DPAPI/TPM hardware binding). No other module moves to C++ without a concrete, measurable, market-relevant justification.

---

## 2. Namespace

**Ásgarðr** — the governing realm. The overarching namespace under which all engine modules live. In Norse mythology, Ásgarðr is the realm of the gods — the highest domain. Every module lives within it.

---

## 3. Kernel Layer

The kernel layer forms the execution core of the engine. These modules handle orchestration, runtime execution, lifecycle management, caching, performance, and configuration.

---

### Óðinn — Overseer & Orchestrator

| Property | Detail |
|:---|:---|
| **Responsibility** | Orchestration and coordination only |
| **Layer** | Kernel |
| **Language** | C# |

Óðinn oversees, directs, and triggers. He is the highest coordination point in the engine — the Allfather who knows the state of all modules and decides what happens when. He never executes directly. When something needs to happen, Óðinn decides and delegates. The execution happens elsewhere.

Task scheduling flows through Óðinn: Dagr provides the time, Óðinn decides what to trigger, Jǫrmungandr executes it.

---

### Jǫrmungandr — Execution Core

| Property | Detail |
|:---|:---|
| **Responsibility** | Runtime execution — the engine that runs processes |
| **Layer** | Kernel |
| **Language** | C# |

Jǫrmungandr is the World Serpent — the core runtime where processes actually run. He does the work; he does not decide who does what. That is Óðinn's role. Jǫrmungandr also handles kernel-level I/O (reading from and writing to disk), as I/O is inseparable from execution at this level. Modules are lazy-loaded through Jǫrmungandr — nothing runs until it is needed.

---

### Yggdrasil — Lifecycle & Dependency Manager

| Property | Detail |
|:---|:---|
| **Responsibility** | Module lifecycle tracking and dependency management |
| **Layer** | Kernel |
| **Language** | C# |

Yggdrasil is the World Tree — the structure that connects all realms and holds the cosmos together. In the engine, Yggdrasil tracks which modules are loaded, manages the boot sequence, and handles module dependencies. It knows the state of every module at all times.

---

### Frigg — Cache

| Property | Detail |
|:---|:---|
| **Responsibility** | In-memory caching of frequently accessed data |
| **Layer** | Kernel |
| **Language** | C# |
| **Status** | LOCKED |

Frigg is static and ready — she holds what has already been retrieved so it does not need to be fetched again. Frigg sits above Jǫrmungandr's I/O: she checks the cache first, and only when the data is not cached does Jǫrmungandr go to disk. This reduces I/O overhead and keeps the engine responsive.

---

### Freyja — Performance Optimization

| Property | Detail |
|:---|:---|
| **Responsibility** | Active performance monitoring and throttling |
| **Layer** | Kernel |
| **Language** | C# |
| **Status** | LOCKED |

Freyja is active and adaptive — she monitors resource usage and throttles where needed to keep the engine within its performance targets. Where Frigg is passive (stored data), Freyja is dynamic (active monitoring). They are distinct modules with no overlap.

---

### Verðandi — Configuration Manager

| Property | Detail |
|:---|:---|
| **Responsibility** | Dynamic loading and management of engine configurations |
| **Layer** | Kernel |
| **Language** | C# |
| **Status** | LOCKED |

Verðandi is one of the three Norns — she governs the present. In the engine she manages the current configuration state: dynamically loading engine settings and variables, providing them to modules that need them. She is separate from Frigg (cache) and Freyja (performance) with no overlap.

---

## 4. Security Layer

The security layer protects the engine, manages cryptography, and stores secrets. Applications never touch this layer directly — all secure operations go through Bifrǫst.

---

### Heimdallr — Gatekeeper

| Property | Detail |
|:---|:---|
| **Responsibility** | Authentication, authorisation, and session management |
| **Layer** | Security |
| **Language** | C# |

Heimdallr stands at the boundary of Ásgarðr and decides who may pass. In the engine he controls all access: authenticating callers, authorising operations, and managing sessions. Creator mode is entered through Heimdallr via an undocumented boot key combination that opens a passphrase prompt. Normal boot proceeds without any prompt.

Heimdallr also establishes the ExecutionContext (Creator / Dev / Other) based on the active license type, and delivers it via Hermóðr to modules that need it — such as Vǫr for the tag gate.

---

### Surtr — Core Protection

| Property | Detail |
|:---|:---|
| **Responsibility** | Kernel integrity and memory protection |
| **Layer** | Security |
| **Language** | C++ |

Surtr guards the kernel against manipulation, exploitation, and unauthorised memory access. Integrity is verified via GCM authentication tags throughout — no HMAC anywhere in the engine. Surtr is one of the three modules implemented in C++ because memory protection requires hardware-level precision that C# cannot guarantee.

---

### Vǫlundr — Cipher

| Property | Detail |
|:---|:---|
| **Responsibility** | All cryptographic operations — the sole key generator |
| **Layer** | Security |
| **Language** | C++ (crypto core) |

Vǫlundr is the master smith of the Norse gods — the ultimate craftsman. In the engine he is the sole source of all keys and all cryptographic operations. AES-256-GCM encryption and decryption, HKDF-SHA256 key derivation, RSA key exchange and signature, key distribution and rotation — all of it flows through Vǫlundr. No other module generates keys. No other module performs cryptographic operations.

Vǫlundr is implemented in C++ for the crypto core because cryptographic operations require explicit memory control — key material must be explicitly zeroed after use, which C# cannot guarantee with its managed memory model.

---

### Níðavöllir — Engine Vault

| Property | Detail |
|:---|:---|
| **Responsibility** | Encrypted secure storage for engine infrastructure secrets |
| **Layer** | Security |
| **Language** | C# |

Níðavöllir is the forge of the dwarves — the place where the most precious things are kept. In the engine it is the vault that holds engine infrastructure secrets: per-app wrapping keys and the app slot index. It never holds app data keys or client data. The vault is a custom binary format, not a database. It is rooted directly by TPM/DPAPI at boot via Níðhǫggr.

**Vault scope:** engine infrastructure secrets only. App key material rests in each app's own vault — never in Níðavöllir. A compromised engine vault leaks nothing about apps.

---

### Níðhǫggr — Native Layer

| Property | Detail |
|:---|:---|
| **Responsibility** | DPAPI/TPM hardware binding — seal and unseal operations |
| **Layer** | Security |
| **Language** | C++ |

Níðhǫggr is the dragon that gnaws at the roots of Yggdrasil — the creature that operates at the deepest level. In the engine he is the native layer that interfaces directly with the Windows operating system security infrastructure: DPAPI and TPM. Keys sealed by Níðhǫggr are machine-bound — the encrypted data is cryptographically useless on any other machine. Níðhǫggr is implemented in C++ because DPAPI/TPM access requires direct OS-native calls.

Formerly known as NativeGuard. Renamed to align with the architectural naming language.

---

## 5. Logging & Observability Layer

The engine maintains two logging domains that never mix. Engine infrastructure activity is logged internally. Application domain activity is logged separately by the application. Together they form a complete and legally defensible audit chain.

---

### Huginn — Internal Telemetry Daemon

| Property | Detail |
|:---|:---|
| **Responsibility** | Internal telemetry — developer-facing system and performance data |
| **Layer** | Logging & Observability |
| **Language** | C# |

Huginn is one of Óðinn's two ravens — Thought. He flies out and returns with information. In the engine Huginn collects internal system and performance data for developer use. He is not exposed to applications. All Creator License sessions are logged separately in a dedicated Huginn channel. Huginn starts in bootstrap mode at the earliest stable point during boot — logging begins immediately and transitions to full operational mode once boot is complete. One continuous log, no gap.

Note: Mímir is cancelled. Huginn absorbed Mímir's former internal logger role.

---

### Muninn — Observability / Public Logger

| Property | Detail |
|:---|:---|
| **Responsibility** | Subscription-based public event stream for applications |
| **Layer** | Logging & Observability |
| **Language** | C# |
| **Status** | LOCKED |

Muninn is Óðinn's second raven — Memory. He carries what has happened. In the engine Muninn is the subscription-based event stream that applications consume via Bifrǫst. Applications subscribe to the events they need; Muninn delivers them. He does not log internally — that is Huginn's domain.

**Event categories (fixed byte values):**

| Category | Value |
|:---|:---|
| Lifecycle | 0x01 |
| Security | 0x02 |
| Audit | 0x03 |
| Performance | 0x04 |
| Error | 0x05 |
| AppEvent | 0x06 |
| License | 0x07 |
| Vault | 0x08 |
| Update | 0x09 |
| Configuration | 0x0A |

---

## 6. Time Layer

---

### Dagr — Timekeeper

| Property | Detail |
|:---|:---|
| **Responsibility** | Time authority — provides verified time to whoever asks |
| **Layer** | Time |
| **Language** | C# |
| **Status** | LOCKED |

Dagr is the Norse personification of day — the bringer of light and time. In the engine Dagr is the sole time authority. He provides verified time to whatever module asks for it. He does not schedule, trigger, or control I/O. He answers one question: *what time is it, and how verified is that time?*

**Two time sources:**

| Source | Type | Use |
|:---|:---|:---|
| **RTC** | Hardware clock, unverified | Non-legal operations, local storage, autosave, build date-stamping when air-gapped |
| **NTP** | Network time, working tier | Standard operations, build date-stamping when connected |
| **NTS** | Network Time Security, attestation tier | Legally-operative acts only — Rules Module signing, audit timestamps |

**The rule:** does this need to hold up before an auditor? Yes → NTS. No → NTP or RTC.

NTS is the only explicit NTS exception in the engine — the Rules Module signing chain. Everything else uses WORKING time (RTC or NTP). NTS never blocks; if unavailable, the legally-operative act fails closed for that action only.

---

## 7. Reliability Layer

---

### Eir — Health Diagnostics & Light Recovery

| Property | Detail |
|:---|:---|
| **Responsibility** | Health monitoring and light fault recovery |
| **Layer** | Reliability |
| **Language** | C# |
| **Status** | LOCKED |

Eir is the Norse goddess of healing. In the engine she monitors system health and recovers light faults and inconsistent states without escalation. RAM usage above 80% triggers garbage collection. Above 95% escalates to Baldr. Eir handles what she can; she knows when to call for help.

---

### Baldr — Serious Fault Restoration

| Property | Detail |
|:---|:---|
| **Responsibility** | Serious fault restoration — three fault categories |
| **Layer** | Reliability |
| **Language** | C# |
| **Status** | LOCKED |

Baldr is the most beloved of the Norse gods — pure, good, and resilient. In the engine he handles serious faults that Eir cannot resolve. Three fault categories: Transient, StateCorruption, SecurityCritical. Each has a defined response path. A FaultReport object carries diagnostic information for planned surfacing via Hermóðr.

---

### Ragnarǫk — Controlled State Transition

| Property | Detail |
|:---|:---|
| **Responsibility** | Controlled shutdown and controlled restore |
| **Layer** | Reliability |
| **Language** | C# |
| **Status** | LOCKED |

In Norse mythology, Ragnarǫk is both the destruction of the world and its subsequent rebirth — cleaner and renewed. In the engine Ragnarǫk owns controlled state transitions: the end and the beginning again.

**Shutdown paths:**
- UnrecoverableFault → exit 1
- LicenseExpired → exit 2

Both paths broadcast EngineCritical via Gjallarhorn before exit.

**Restore:** when the engine needs to come back from a critical failure state, Ragnarǫk manages the controlled restore — the world reborn. This is distinct from Eir's light recovery and Baldr's serious fault restoration; those are in-process recoveries. Ragnarǫk handles transitions that take the engine fully down and back up.

---

### Mjǫlnir — Secure Update Manager

| Property | Detail |
|:---|:---|
| **Responsibility** | Receiving and applying verified update payloads |
| **Layer** | Reliability |
| **Language** | C# |

Mjǫlnir is the hammer of Thor — forged by the dwarven brothers, the ultimate instrument of precise, powerful action. In the engine Mjǫlnir receives verified, signed, timestamped update payloads and applies them. That is his only job.

Mjǫlnir does not sign (Vǫlundr), verify signatures (Vǫlundr/Surtr), timestamp (Dagr), carry messages (Hermóðr), gate authority (Heimdallr), or shut down on failure (Ragnarǫk). Each of those belongs to its rightful module. Mjǫlnir strikes the update into place — nothing else.

---

## 8. Integration & Governance Layer

---

### Bifrǫst — Liaison Bridge

| Property | Detail |
|:---|:---|
| **Responsibility** | Sole entry point for applications to the engine |
| **Layer** | Integration |
| **Language** | C# |
| **Status** | LOCKED |

Bifrǫst is the rainbow bridge connecting Ásgarðr to the other realms. In the engine Bifrǫst is the only entry point through which applications communicate with the engine. Applications never reach into the engine directly. Every call goes through Bifrǫst.

**Three-state feature model:** Active / Inactive / Unavailable. Applications declare their active and inactive features at registration. The engine enforces this; it never hardcodes tier rules. The application decides what features are available per license tier.

**IAppMessenger** is the mandatory interface every MÚSpel application must implement. The engine calls this interface without knowing which application is behind it.

---

### Hermóðr — Point-to-Point Courier

| Property | Detail |
|:---|:---|
| **Responsibility** | Point-to-point message delivery between modules |
| **Layer** | Integration |
| **Language** | C# |
| **Status** | LOCKED |

Hermóðr is the messenger of the gods — swift, trusted, and precise. In the engine Hermóðr carries messages from one specific module to one specific other module. He does not broadcast; that is Gjallarhorn's role. He does not know what is in the message; he only delivers it.

**MessageType:** Query / Command / Ping / Acknowledgement  
**DeliveryFailureReason:** DestinationNotFound / HandlerFailed / InvalidMessage / Timeout  
**Reply:** Passed() or Failed()

---

### Gjallarhorn — Event Broadcaster

| Property | Detail |
|:---|:---|
| **Responsibility** | One-to-many event broadcasting |
| **Layer** | Integration |
| **Language** | C# |
| **Status** | LOCKED |

Gjallarhorn is the horn of Heimdallr — when it sounds, all realms hear it. In the engine Gjallarhorn broadcasts events to all subscribers simultaneously. Where Hermóðr is point-to-point, Gjallarhorn is one-to-many. EngineCritical events (from Ragnarǫk) travel via Gjallarhorn so all subscribers receive the signal before shutdown.

---

### Urðr — Immutable Audit Logger

| Property | Detail |
|:---|:---|
| **Responsibility** | Append-only immutable recording of legally-mandated auditable acts |
| **Layer** | Compliance |
| **Language** | F# |

Urðr is one of the three Norns — specifically the Norn of the past. What Urðr has woven cannot be unwoven. What she has recorded cannot be changed. In the engine she is the immutable audit logger — the sole recorder of legally-mandated auditable acts.

**Urðr records exactly three things and nothing else:**
- Calculations — the math must be traceable and reproducible
- Rulesets — which rules governed which calculation, in which fiscal year
- NTS-stamped legally-operative acts — Belastingdienst submissions, ruleset updates, signed calculations

The record is append-only — entries can only be added, never modified or deleted. Each entry contains a cryptographic hash of the previous entry, forming an unbreakable chain. Tampering with any entry breaks the chain mathematically. Vǫlundr signs each entry; Dagr embeds an NTS timestamp into each legally-operative act entry.

Urðr is the first F# module inside the engine — justified by F#'s immutability-by-default properties, which are the right tool for an immutable record. AOT compatibility must be verified during the build phase.

Urðr is not a general logger. The engine logs infrastructure activity via Huginn for operational and debugging purposes. Muninn provides the public event stream for applications. Neither of those is a legal compliance requirement. Urðr is the only module whose output is legally mandated.

---

### Forseti — Compliance Gate

| Property | Detail |
|:---|:---|
| **Responsibility** | Pre-execution compliance gate for legally-operative acts |
| **Layer** | Governance |
| **Language** | C# |

Forseti is the Norse god of justice and law — the best judge among gods and men. In the engine Forseti is the event-triggered gate that activates before any legally-operative act. He judges compliance; security enforces. The pipeline for a legally-operative act: Forseti gates → Dagr stamps NTS → Vǫlundr signs → out.

---

### Vǫr — Version & Compatibility Authority

| Property | Detail |
|:---|:---|
| **Responsibility** | Version knowledge and compatibility decision |
| **Layer** | Governance |
| **Language** | C# |

Vǫr is the Norse goddess of wisdom and awareness — she knows all things. In the engine Vǫr is the version authority. She knows the engine's own compiled-in version, judges whether a connecting application is compatible, and applies the maturity tag gate. Bifrǫst is the transport; Vǫr is the decision-maker.

**Vǫr never touches license validity** — that is Týr's domain. Versioning and licensing are orthogonal and never consult each other.

The third-party developer discovery API (capability listing via Bifrǫst) is a future Phase 5 addition, built when a Development License customer exists.

---

## 9. Module Summary

| Module | Layer | Responsibility | Language |
|:---|:---|:---|:---|
| Óðinn | Kernel | Orchestration & coordination | C# |
| Jǫrmungandr | Kernel | Execution core & kernel I/O | C# |
| Yggdrasil | Kernel | Lifecycle & dependency management | C# |
| Frigg | Kernel | Cache | C# |
| Freyja | Kernel | Performance optimisation | C# |
| Verðandi | Kernel | Configuration management | C# |
| Heimdallr | Security | Authentication, authorisation, session management | C# |
| Surtr | Security | Kernel integrity & memory protection | C++ |
| Vǫlundr | Security | All cryptographic operations — sole key generator | C++ |
| Níðavöllir | Security | Engine vault — encrypted infrastructure secret storage | C# |
| Níðhǫggr | Security | DPAPI/TPM hardware binding | C++ |
| Huginn | Logging | Internal telemetry daemon | C# |
| Muninn | Logging | Public observability event stream | C# |
| Dagr | Time | Time authority (RTC / NTP / NTS) | C# |
| Eir | Reliability | Health diagnostics & light recovery | C# |
| Baldr | Reliability | Serious fault restoration | C# |
| Ragnarǫk | Reliability | Controlled state transition (shutdown & restore) | C# |
| Mjǫlnir | Reliability | Secure update application | C# |
| Bifrǫst | Integration | Sole application entry point | C# |
| Hermóðr | Integration | Point-to-point message courier | C# |
| Gjallarhorn | Integration | One-to-many event broadcaster | C# |
| Urðr | Compliance | Immutable audit logger — legally-mandated acts only | F# |
| Forseti | Governance | Pre-execution compliance gate | C# |
| Vǫr | Governance | Version & compatibility authority | C# |

---

## 10. Cancelled Modules

| Module | Status | Reason |
|:---|:---|:---|

---

*© Era Digital 2025–2026 | Eric Slangen | Confidential | ED-MSPL-MODL-001 | Version 1.0*
