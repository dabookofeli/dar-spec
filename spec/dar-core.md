# Digital Action Receipts (DAR) — Core Specification

## Status of This Document

This document defines the core, mandatory fields and invariants of a Digital Action Receipt (DAR).

This specification is **normative** unless otherwise stated.  
Extensions MAY be defined separately but MUST NOT alter the semantics of this core specification.

---

## 1. Introduction

A Digital Action Receipt (DAR) is a cryptographically verifiable record asserting that a specific digital action occurred at a defined point in time.

DARs are designed to provide:
- Evidence of occurrence
- Portability across systems
- Independent verification
- Long-term auditability

DARs do **not** assert correctness, intent, compliance, or authorization.  
They assert only that an action occurred and was recorded by an issuer.

---

## 2. Terminology

The key words MUST, MUST NOT, REQUIRED, SHALL, SHOULD, MAY, and OPTIONAL in this document are to be interpreted as described in RFC 2119.

- **Action**: A discrete digital event with a defined commit point.
- **Commit point**: The moment an action becomes finalized within an issuer system.
- **Issuer**: The system that generates a DAR.
- **Verifier**: Any independent party validating a DAR.
- **Canonical form**: A deterministic serialization of a DAR payload.
- **Receipt hash**: A cryptographic hash representing the canonicalized DAR payload.

---

## 3. Design Principles (Non-Normative)

DARs are intentionally:
- Minimal
- Vendor-neutral
- Append-only
- Non-judgmental

DARs are not logs, policies, or compliance engines.

---

## 4. Digital Action Model

A DAR MUST correspond to a single digital action.

An action MUST have:
- A clearly defined commit point
- A unique action identifier
- A timestamp representing when the action was finalized

DARs MUST be generated at or immediately after the commit point.

---

## 5. Object Model

A DAR consists of:
1. a **payload** (canonicalized, hashed, and optionally signed)
2. an OPTIONAL **envelope** (transport metadata such as signature, locator, or derived receipt hash)

### 5.1 Core DAR Payload (Normative)

The payload is the object that MUST be canonicalized and hashed.

```json
{
  "dar_version": "1.0",
  "action_type": "string",
  "action_id": "string",
  "timestamp": "RFC3339 string",
  "actor": {
    "type": "system | service | agent | user",
    "id": "string"
  },
  "inputs_hash": "string",
  "outcome_hash": "string"
}
5.2 Field Definitions (Normative)
Field	Requirement
dar_version	MUST identify the DAR core version (e.g., "1.0")
action_type	MUST describe the category of action
action_id	MUST uniquely identify the action within the issuer’s namespace
timestamp	MUST represent the action commit time in RFC3339 format
actor.type	MUST indicate the actor class
actor.id	MUST be an opaque identifier
inputs_hash	MUST be a hash of referenced inputs
outcome_hash	MUST be a hash of referenced outcomes

Note (Non-Normative):
action_id uniqueness is intentionally issuer-scoped.

5.3 Envelope (Normative)
A DAR MAY be transported inside an envelope object that contains the core payload and optional metadata.

json
Copy code
{
  "payload": { "...": "DAR payload object..." },
  "signature": {
    "algorithm": "string",
    "key_id": "string",
    "value": "base64-encoded signature"
  },
  "receipt_hash": "string",
  "locator": "string"
}
Canonicalization, hashing, and signing MUST be performed over the payload only.

6. Data Handling Requirements (Normative)
DAR payloads MUST NOT embed raw input or output data.

Hashes MUST reference externally stored artifacts.

Sensitive or personally identifiable data SHOULD NOT appear in the DAR payload.

7. Canonicalization (Normative)
Canonicalization applies to the DAR payload only.

The canonical form MUST:

Use UTF-8

Sort object keys lexicographically

Normalize timestamps

Be deterministic

8. Receipt Hash Generation (Normative)
The receipt hash MUST be computed as:

scss
Copy code
SHA-256(canonical_payload_bytes)
9. Optional Cryptographic Signatures (Normative)
If present, signatures MUST be computed over the canonicalized payload bytes.

Signatures assert issuance authenticity, not action correctness.

10. Storage Requirements (Normative)
DARs MUST be stored immutably after issuance.

11. Verification (Normative)
Verification MUST be possible without issuer cooperation and include:

Payload retrieval

Canonicalization

Hash recomputation

Signature validation (if present)

12. Extensions (Normative)
Extensions MUST NOT modify core semantics or invalidate existing DARs.

13. Non-Goals (Normative)
DARs do NOT:

Judge correctness

Enforce compliance

Replace audits

Replace system logs
