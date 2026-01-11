# Digital Action Receipts (DAR) — Core Specification
## Status of This Document

This document defines the core, mandatory fields and invariants of a Digital Action Receipt (DAR).

This specification is normative unless otherwise stated.
Extensions MAY be defined separately but MUST NOT alter the semantics of this core specification.

## 1. Introduction

A Digital Action Receipt (DAR) is a cryptographically verifiable record asserting that a specific digital action occurred at a defined point in time.

DARs are designed to provide:
- Evidence of occurrence
- Portability across systems
- Independent verification
- Long-term auditability

DARs do not assert correctness, intent, compliance, or authorization.
They assert only that an action occurred and was recorded.

## 2. Terminology

The key words MUST, MUST NOT, REQUIRED, SHALL, SHOULD, MAY, and OPTIONAL in this document are to be interpreted as described in RFC 2119.

- Action: A discrete digital event with a defined commit point.
- Issuer: The system that generates a DAR.
- Verifier: Any independent party validating a DAR.
- Canonical Form: A deterministic serialization of a DAR payload.
- Receipt Hash: A cryptographic hash representing the DAR payload.

## 3. Design Principles (Non-Normative)

DARs are intentionally:

- Minimal
- Vendor-neutral
- Append-only
- Non-judgmental

DARs are not logs, policies, or compliance engines.

## 4. Digital Action Model

A DAR MUST correspond to a single digital action.

An action MUST have:

- A clearly defined commit point
- A unique action identifier
- A timestamp representing when the action was finalized

DARs MUST be generated at or immediately after the commit point.

## 5. Core DAR Payload

A DAR payload MUST include the following fields:

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

## 5.1 Field Definitions
Field	- Requirement
dar_version -	MUST identify the DAR core version
action_type -	MUST describe the category of action
action_id -	MUST uniquely identify the action
timestamp -	MUST represent the action commit time
actor.type -	MUST indicate the actor class
actor.id -	MUST be an opaque identifier
inputs_hash -	MUST hash referenced inputs
outcome_hash -	MUST hash referenced outcomes

## 6. Data Handling Requirements

- DAR payloads MUST NOT embed raw input or output data.
- Hashes MUST reference externally stored artifacts.
- Personally identifiable or sensitive data SHOULD NOT appear in the DAR payload.

## 7. Canonicalization

Before hashing or signing, DAR payloads MUST be canonicalized.

Canonicalization MUST:

- Serialize using UTF-8
- Sort object keys lexicographically
- Normalize timestamps
- Exclude non-deterministic fields

The canonical form MUST produce identical output for identical logical content.

## 8. Receipt Hash Generation

A Receipt Hash MUST be computed over the canonicalized DAR payload.

receipt_hash = SHA-256(canonical_dar_payload)

The receipt hash SHALL serve as the primary identifier for the DAR.

## 9. Optional Cryptographic Signatures

DARs MAY include a cryptographic signature.

If present, a signature MUST include:

{
  "algorithm": "string",
  "key_id": "string",
  "value": "base64-encoded signature"
}


Signatures assert issuance authenticity, not action correctness.

## 10. Storage Requirements

DARs MUST be stored in an append-only manner.

Permissible storage systems include:

- Immutable object storage
- Append-only logs
- Content-addressed storage
- Customer-controlled repositories

DARs MUST NOT be modified or deleted after issuance.

## 11. Referencing and Retrieval

Systems SHOULD reference DARs by:

- Receipt hash
- Optional locator or URI

DARs SHOULD be retrieved on demand, not pushed downstream.

## 12. Verification

Verification of a DAR MUST be possible without privileged access.

Verification steps include:

- Retrieve DAR payload
- Canonicalize payload
- Recompute receipt hash
- Validate signature (if present)
- Validate referenced hashes (if available)

Verification MUST NOT require cooperation from the issuer.

## 13. Extensions

Extensions:

- MUST be explicitly versioned
- MUST NOT modify core field semantics
- MUST NOT invalidate existing DARs

Examples include:

- Multi-party approvals
- AI decision metadata
- Zero-knowledge attestations

## 14. Non-Goals

DARs explicitly DO NOT:

- Judge correctness
- Enforce compliance
- Prove authorization
- Replace audits
- Replace system logs

DARs answer only:

Did this digital action occur, and can it be proven later?

## 15. Backward Compatibility

Future versions of DAR MUST:

- Preserve existing field semantics
- Maintain verifiability of historical DARs
- Avoid mandatory field removal

## 16. Security Considerations

DAR security depends on:

- Cryptographic hash integrity
- Canonicalization correctness
- Storage immutability

Compromise of issuer systems MAY affect issuance but MUST NOT prevent independent verification of existing DARs.

## 17. IANA Considerations

This document defines no IANA registries.

## 18. Conclusion

Digital Action Receipts provide a foundational evidence primitive for digital systems operating at scale.

DARs enable long-term trust without requiring long-term trust in any single system.
