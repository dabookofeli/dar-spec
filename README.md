# Digital Action Receipts (DAR) — Specification

Digital Action Receipts (DAR) define a minimal, vendor-neutral format for issuing cryptographically verifiable proof that a digital action occurred.

DARs are:
- **Append-only**
- **Portable**
- **Non-judgmental**
- **Independently verifiable**

This repository contains the **canonical DAR specification**.

---

## Status

**Status:** Draft (v0.x)  
**Stability:** Backwards compatibility is **not guaranteed** until v1.0.

---

## What DAR is (and is not)

**DAR is:**
- A small signed artifact + hashed references (IDs/hashes, not sensitive payloads)
- A portable proof object that can be verified independently

**DAR is not:**
- A compliance judgment or “truth machine”
- A required storage system (receipts can live anywhere)
- A mandate to reveal private data (use references/hashes)

---

## Specification

- **DAR Core Specification:** [`spec/dar-core.md`](spec/dar-core.md)

---

## Minimal example (illustrative)

```json
{
  "dar_version": "1.0",
  "action_type": "ai_decision",
  "timestamp": "2026-01-10T02:15:00Z",
  "actor": "agent:billing-bot-v3",
  "context_hash": "sha256:abc123...",
  "outcome_hash": "sha256:def456...",
  "prev_receipt_hash": "sha256:0000...",
  "signature": "ed25519:..."
}
