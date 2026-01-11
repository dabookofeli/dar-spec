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
- **Commit point**: The moment an action becomes finalized within an issuer system (e.g., transaction committed, approval recorded).
- **Issuer**: The system that generates a DAR.
- **Verifier**: Any independent party validating a DAR.
- **Canonical form**: A deterministic serialization of a DAR payload.
- **Receipt hash**: A cryptographic hash representing the canonicalized DAR payload.

---

## 3. Design Principles (Non-Normative)

DARs are intentionally:
- Minimal
- Vendor-neutral
- Append-only (immutability-oriented)
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
```

### 5.2 Field Definitions (Normative)
| Field          | Requirement                                                     |
| -------------- | --------------------------------------------------------------- |
| `dar_version`  | MUST identify the DAR core version (e.g., `"1.0"`)              |
| `action_type`  | MUST describe the category of action (issuer-defined string)    |
| `action_id`    | MUST uniquely identify the action within the issuer’s namespace |
| `timestamp`    | MUST represent the action commit time in RFC3339 format         |
| `actor.type`   | MUST indicate the actor class                                   |
| `actor.id`     | MUST be an opaque identifier (no required semantics)            |
| `inputs_hash`  | MUST be a hash of referenced inputs (see §6)                    |
| `outcome_hash` | MUST be a hash of referenced outcomes (see §6)                  |

### Note (Non-Normative):
action_id uniqueness scope is intentionally issuer-scoped to avoid requiring global coordination.

### 5.3 Envelope (Normative)

A DAR MAY be transported inside an envelope object that contains the core payload and optional metadata.

The envelope MAY include:

- a cryptographic signature
- a derived receipt hash
- optional retrieval or locator hints

The envelope MUST include the payload field.

```json
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
```

### Normative requirements:

- Canonicalization, receipt hash computation, and signature generation MUST be performed over the payload object only.
- Envelope fields MUST NOT be included in canonicalization, hashing, or signing.
- Envelope fields MUST NOT affect the semantic interpretation of the payload.

## 6. Data Handling Requirements (Normative)

- DAR payloads MUST NOT embed raw input or output data.
- Hash fields MUST reference externally stored artifacts (or an empty artifact set; see below).
- Personally identifiable or sensitive data SHOULD NOT appear in the DAR payload.

### 6.1 Empty Artifact Sets (Normative)

If an action has no inputs or no outcomes, inputs_hash or outcome_hash MUST still be present and MUST be set to a well-defined value.

The RECOMMENDED value is the SHA-256 hash of the empty byte string:

```nginx
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

Implementations MAY use an equivalent well-defined empty-set hash but MUST be consistent within an issuer.

## 7. Canonicalization (Normative)

Canonicalization applies to the DAR payload only, excluding all envelope fields.

Before hashing or signing, DAR payloads MUST be canonicalized.

Canonicalization MUST:

- Serialize using UTF-8
- Sort object keys lexicographically
- Normalize timestamps to an equivalent RFC3339 string form
- Exclude non-deterministic fields (none exist in the core payload)

The canonical form MUST produce identical output for identical logical content.

Recommendation (Non-Normative):
Implementations SHOULD follow RFC 8785 (JSON Canonicalization Scheme / JCS) to avoid divergence.

## 8. Receipt Hash Generation (Normative)

A receipt hash MUST be computed over the canonicalized DAR payload:

```ini    
receipt_hash = SHA-256(canonical_payload_bytes)
```

The receipt hash SHALL serve as the primary identifier for the receipt when referencing or retrieving DARs.

Note (Normative):

- The receipt hash is derived and does not need to be included in the payload.
- Security assumes the collision resistance of the underlying hash function.

## 9. Optional Cryptographic Signatures (Normative)

A DAR MAY include a cryptographic signature over the canonicalized payload bytes.

If present, the signature block MUST include:

```json
{
  "signature": {
    "algorithm": "string",
    "key_id": "string",
    "value": "base64-encoded signature"
  }
}
```

Signatures assert issuance authenticity, not action correctness.

Normative requirement:
The signature MUST be computed over the canonicalized payload bytes (not over the envelope).

## 10. Storage Requirements (Normative)

DARs MUST be stored in a manner that preserves immutability after issuance.

Permissible storage systems include:

- Immutable object storage
- Append-only logs
- Content-addressed storage
- Customer-controlled repositories

Issuers MUST NOT modify a stored DAR payload after issuance.

### Note (Non-Normative):
Physical deletion may be possible in some systems; “append-only” means issuers treat issued receipts as immutable artifacts.

## 11. Referencing and Retrieval (Normative)

Systems SHOULD reference DARs by:

- Receipt hash (primary)
- Optional locator or URI (secondary)

DARs SHOULD be retrieved on demand, not pushed downstream.

## 12. Verification (Normative)

Verification of a DAR MUST be possible without privileged access.

A verifier MUST be able to:

- Retrieve the DAR payload
- Canonicalize the payload
- Recompute the receipt hash
- Validate the signature (if present)

A verifier SHOULD validate referenced hashes (inputs/outcomes) if the referenced artifacts are available.

Verification MUST NOT require cooperation from the issuer for already-issued receipts.

## 13. Extensions (Normative)

Extensions:

- MUST be explicitly versioned
- MUST NOT modify core field semantics
- MUST NOT invalidate existing DARs

Examples (Non-Normative):

- Multi-party approvals
- AI decision metadata
- Zero-knowledge attestations

## 14. Non-Goals (Normative)

DAR (core) explicitly DOES NOT:

- Judge correctness
- Enforce compliance
- Prove authorization
- Replace audits
- Replace system logs

DARs answer only:

Did this digital action occur, and can it be proven later?

## 15. Backward Compatibility (Normative)

Future versions of DAR MUST:

- Preserve existing field semantics
- Maintain verifiability of historical DARs
- Avoid mandatory field removal

## 16. Security Considerations (Normative)

DAR security depends on:

- Cryptographic hash integrity
- Canonicalization correctness
- Storage immutability

Compromise of issuer systems MAY affect issuance going forward but MUST NOT prevent independent verification of already-issued receipts.

## 17. IANA Considerations

This document defines no IANA registries.

## 18. Conclusion

Digital Action Receipts provide a foundational evidence primitive for digital systems operating at scale.

DARs enable long-term trust without requiring long-term trust in any single system.
