# MÚSpel Engine — Compliance Architecture & Legal Framework

*Modular Unified System Program Execution Layer — Compliance Architecture*

*****CONFIDENTIAL*****

**Project:** MÚSpel Engine  
**Document Version:** 1.0  
**DID:** ED-MSPL-CMPL-001  
**Date:** 02 June 2026  
**Author:** Eric Slangen  
**Proprietary to:** Era Digital  
**© 2025–2026 Era Digital. All rights reserved.**

**Status:** Current

---

## 1. Purpose

This document describes the compliance architecture of the MÚSpel Engine — how legally-operative acts are timestamped, governed, and made auditable, and what obligations the engine imposes on itself and on applications built on top of it.

The target market includes Dutch accountant agencies and potentially governmental institutions. Compliance is not optional and not cosmetic. Every legally-operative act must be traceable, timestamped with legally defensible time, and signed by the appropriate authority.

---

## 2. The Compliance Chain

Four modules form the compliance backbone of the engine:

**Dagr** — the time authority. Provides the NTS-verified timestamp that makes a legally-operative act defensible before an auditor.

**Vǫlundr** — the judicial seal. Signs the act with a cryptographic signature that proves it is authentic and untampered.

**Forseti** — the judge. Determines whether the act is complete, correct, and lawful. If not satisfied, the legally-operative act is rejected. Either way, the outcome is permanently recorded — a rejection is in itself a legally defensible event.

**Urðr** — the court clerk. Appends the signed, timestamped, judged act to the permanent, cryptographically-chained record. What Urðr records cannot be altered.

The pipeline for any legally-operative act:

```
Dagr (NTS timestamp — when did this happen, provably?)
    ↓
Vǫlundr (judicial seal — cryptographic signature, prove it is authentic and untampered)
    ↓
Forseti (judge — is this act complete, correct, and lawful? May it be permanently recorded?)
    ↓
Urðr (court clerk — append to the permanent, chained audit log)
    ↓
Out — outcome permanently recorded, legally defensible whether approved or rejected
```

---

## 3. Dagr — The Time Authority

Dagr is the Norse god of day and time — he rides his chariot across the sky, marking the passage of time with certainty and order. In the engine he is the time authority: the sole provider of verified, legally defensible timestamps. Every legally-operative act that must hold up before an auditor, a regulator, or a court depends on Dagr to mark exactly when it happened — provably and irrefutably.

Dagr does not decide whether an act may proceed — that is Forseti's role. Dagr does one thing: he answers the question "when?", with proof.

### 3.1 Time Tiers

Dagr provides time at two distinct tiers. Using the wrong tier for a given operation is a design error.

| Tier | Source | When to Use |
|:---|:---|:---|
| **WORKING** | RTC (air-gapped) or NTP (connected) | Local storage, autosave, non-legal operations, build date-stamping |
| **ATTESTATION** | NTS — Network Time Security | Any legally-operative act that must hold up before an auditor, Ruleset updates |

**The rule:** does this operation need to hold up before an auditor, a regulator, or a court? Yes → ATTESTATION (NTS). No → WORKING (RTC or NTP).

### 3.2 NTS Sources

Primary: `ntp.time.nl` (Dutch national NTP service, SIDN Labs, Stratum 1, atomic clock verified)  
Secondary: `ntp.vsl.nl` (Van Swinden Laboratorium, Dutch national metrology institute, independent)  
Fallback: FAIL CLOSED for that action only — the legally-operative act does not proceed without a verified timestamp.

NTS never blocks the engine. It fails only the specific act that required it.

### 3.3 NTS Mandated Use Cases

NTS attestation time is mandated in exactly two places in the engine:

**The compliance pipeline** — every legally-operative act that passes through Dagr → Vǫlundr → Forseti → Urðr requires an NTS-verified timestamp. The timestamp is the legal proof of when the act occurred. Without it, the act is not legally defensible.

**The Ruleset signing chain** — when a new Ruleset is signed and distributed, it must carry an NTS timestamp. This is the moment a new set of fiscal laws is formally enacted. That moment must be traceable to an atomic clock source and inseparable from the signed package.

All other time needs in the engine are served by WORKING time (RTC or NTP). NTS is a heavyweight mechanism used only where it earns its place — where the timestamp must be legally defensible. Overusing NTS would add unnecessary dependency and complexity.

### 3.4 NTS Timestamp Embedding

When Dagr provides an NTS timestamp for a legally-operative act, that timestamp is embedded into the signed artifact. It is not a log entry that can be separated from the artifact — it is part of what is signed. The timestamp and the content are inseparable.

---

## 4. Vǫlundr — The Judicial Seal

Vǫlundr is the master smith of Norse mythology — the forger of the finest artifacts, whose work cannot be replicated or undone. In the engine he is the judicial seal: the sole cryptographic authority in the compliance pipeline. He receives a legally-operative act, already timestamped by Dagr, and binds its content, origin, and moment of creation into a single tamper-proof artifact using an RSA cryptographic signature.

Vǫlundr does not decide whether an act may proceed — that is Forseti's role. He does not determine when an act occurred — that is Dagr's role. Vǫlundr does one thing: he proves the act is authentic and untampered. A single altered byte breaks signature verification immediately and irrecoverably. That proof travels with the artifact permanently — it cannot be separated from what it seals.

The full cryptographic architecture of Vǫlundr — key management, AES-256-GCM, HKDF, memory zeroing — is documented in ED-MSPL-SECU-001. Only the compliance-relevant signing responsibility is covered here.

---

## 5. Forseti — The Compliance Gate

Forseti is the Norse god of justice — the most benevolent judge among gods and men. In the engine he is the compliance judge: he verifies that a legally-operative act has been properly timestamped by Dagr, cryptographically sealed by Vǫlundr, and that all security conditions have been met. Only then does he pronounce his ruling — and that ruling, whether approval or rejection, is handed to Urðr for permanent recording.

---

## 6. Urðr — The Immutable Audit Logger

Urðr is the Norn of the past — what she has woven cannot be unwoven. In the engine she is the immutable audit logger: the sole recorder of the three things the law actually requires to be traceable.

**Urðr records exactly three things:**
- Calculations — the math must be traceable and reproducible
- Rulesets — which rules governed which calculation, in which fiscal year
- NTS-stamped legally-operative acts — Belastingdienst submissions, Ruleset updates, signed calculations, including the NTS timestamp verification result and whether it was valid or invalid

**Immutability mechanics:**
- Append-only file access — the OS refuses any operation that modifies existing content
- Cryptographic chaining — each entry contains a hash of the previous entry. Tampering with any entry breaks the chain mathematically and is immediately detectable
- Vǫlundr signs each entry — a tampered entry breaks signature verification
- Dagr NTS timestamp is embedded into each legally-operative entry — inseparable from the content

Urðr is the first F# module inside the engine, justified by F#'s immutability-by-default properties. Native AOT compatibility must be verified at build time.

**Urðr is not a general logger.** The engine logs infrastructure activity via Huginn for operational, development and debugging purposes. Muninn provides the public event stream for applications. Neither of these is a legal compliance requirement. Urðr is the **only** module whose output is legally mandated.

---

## 7. Ruleset — The Fiscal Law Book

### 7.1 What It Is

The Ruleset is an F# module that lives on the application side (BAAS), not in the engine. It holds the fiscal rules, tax rates, thresholds, and other legally-operative parameters for a specific fiscal year and jurisdiction.

It is separate from the BAAS Calculus engine (which does the mathematics). The Calculus module is the calculator; the Ruleset is the law book the calculator reads from.

### 7.2 Versioning — A Justified Exception

The Ruleset is the only module in the entire system that versions separately from its parent unit. This is a deliberate, justified exception to the one-unit versioning rule.

**Reason:** an auditor, regulator, or court must be able to identify exactly which fiscal rules governed a specific calculation — independently of what version of the application or engine was running at the time. The application version tells you nothing about which tax rates were active. Only the Ruleset version tells you that.

**Version scheme:** `DD.MM.YYYY(fiscal).HHMM`

| Field | Meaning |
|:---|:---|
| DD.MM | Day and month the rules were signed and pushed |
| YYYY(fiscal) | The fiscal year these rules govern — the critical field |
| HHMM | Time of signing, to the minute |

The fiscal year field is the primary identifier. `2026` means these rules govern fiscal year 2026, regardless of when they were built or deployed.

The version is derived entirely from Dagr's NTS timestamp at the exact moment of signing. No manual versioning is permitted. The Developer or accountant does not set the version number — Dagr mints it, automatically and irrefutably, as part of the signing event.

### 7.3 The Four Auditable Facts

Every signed Ruleset package carries four independently auditable facts:

| Fact | Source | Proves |
|:---|:---|:---|
| CA certificate | External Certificate Authority (Boosten's cert) | Boosten is a verified, registered entity |
| Vǫlundr RSA signature | Internal signing | The package has not been tampered with in transit |
| NTS timestamp | Dagr — attestation tier | When it was signed, legally defensible |
| Rules version number | The version field | Which fiscal year's rules this package governs |

These four facts are independent. Each can be verified separately. Together they make the package legally defensible.

### 7.4 The Signing Chain

When Boosten's accountant prepares a Ruleset update, the chain is:

```
Boosten's accountant edits rates via the Ruleset Update Tool (BAAS Certified — Master License only)
    ↓
Vǫlundr signs the package cryptographically
    ↓
Dagr provides the NTS timestamp — embedded into the signed package
    ↓
Heimdallr gates the operation (Master License only — no other tier may push rules)
    ↓
Hermóðr carries the signed package point-to-point through the engine
    ↓
Mjǫlnir receives the verified payload and applies the update
    ↓
Each receiving engine verifies the full chain before applying — tampered or unsigned payload is refused
```

### 7.5 Historical Integrity

Ruleset versions are never replaced — they are superseded. A calculation performed under the 2025 rules can always be re-verified against the 2025 Ruleset, even when the 2026 rules are active. Historical calculations remain reproducible and auditable.

---

## 8. AVG / GDPR Obligations

### 8.1 Private Data Is Always Locked

Private client information is locked for any third party — including Era Digital. This is a non-negotiable architectural constraint derived from AVG/GDPR.

Developer mode never decrypts personal data. Code fixes and debugging happen at the infrastructure level, not at the data level. Liability is clear: code is Era Digital's responsibility; data is the accountant's responsibility.

### 8.2 Bug Report Bundle

When a bug report is generated, a just-in-time disclosure is shown listing exactly what system data is gathered:
- Fault information
- Version and license data
- Log integrity data
- Device data (OS, installation ID, username)

What is explicitly NOT gathered:
- Any client data belonging to any party — financial, personal, or otherwise

Pressing Send constitutes informed consent. A link to the full privacy statement is provided. Final wording and legal basis require DPO review before production use.

### 8.3 Data Sovereignty

All client data resides exclusively on the machine or server of the party it belongs to. It never reaches Era Digital infrastructure and never enters a cloud service. This is a structural guarantee, not a policy statement. The data cannot leave because the architecture does not provide a path for it to leave.

Sovereignty over client data is not a feature — it is the foundation. AVG/GDPR and Wdo compliance follows naturally from the architecture, not from policy.

---

## 9. Compliance for Application Developers

Applications built on the MÚSpel Engine inherit the engine's compliance infrastructure but carry their own domain obligations.

**What the engine provides:**
- An immutable, cryptographically-chained audit record for legally-mandated acts (Urðr)
- NTS timestamping for legally-operative acts (Dagr)
- Cryptographic signing of critical artifacts (Vǫlundr)
- A pre-execution compliance gate (Forseti)
- Machine-bound encrypted storage (Níðhǫggr + Níðavöllir)
- Infrastructure and debug logging for operational purposes (Huginn) — not a legal compliance requirement
- A public event stream for application-level observability (Muninn) — not a legal compliance requirement

**What the application is responsible for:**
- Domain-specific compliance rules (e.g. the Ruleset in BAAS)
- Correct use of the engine's compliance chain for legally-operative acts
- AVG/GDPR compliance for client/external data it handles
- The accuracy of the calculations it performs

The MÚSpel Engine is a framework — it knows nothing about what any application does, handles, or is legally obligated to. It provides the infrastructure for compliance to be built upon. What an application does with that infrastructure — the domain rules it enforces, the data it handles, the obligations it carries — is entirely the application's own responsibility. The framework enables. The application commits. The developer decides.

---


---

*© Era Digital 2025–2026 | Eric Slangen | Confidential | ED-MSPL-CMPL-001 | Version 1.0*
