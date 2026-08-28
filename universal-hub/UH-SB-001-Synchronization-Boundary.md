# UH-SB-001 — Universal Hub Synchronization Boundary

**Artifact ID:** UH-SB-001  
**Title:** Universal Hub Synchronization Boundary  
**Format:** Markdown  
**Location:** terra-gaia-framework repository  
**State:** DRAFT  
**Implementation:** NOT_STARTED  
**Canon Promotion:** BLOCKED  
**Immersive Surface:** BLOCKED  
**Depends On:** Promotion Authority Registry (to be defined), Claim Schema, Audit Event Schema, Evidence/Provenance Schema  

---

## Governing Invariant

A projection may preserve authority, but it may never manufacture authority.

No projection, interface, agent, immersive surface, transformation, derivation, aggregation, translation, summarization, visualization, metadata modification, change of storage location, synchronization event, caching, or AI-generated narration may increase the epistemic or canonical authority of its source material.

A synchronization event is never a canon-promotion event.

---

## UH-SB-001.1 — Authority Non-Substitution

No Universal Hub component, projection, interface, agent, visualization, narration, aggregation, transformation, synchronization event, or user engagement may acquire, infer, inherit, or manufacture authority over any claim.

**Explicit extension (Transformation / Derivation Non-Promotion):**  
No transformation, derivation, aggregation, translation, summarization, visualization, metadata modification, or change of storage location may increase the epistemic or canonical authority of its source material.

A system must not be able to take an unauthorized claim and make it appear authorized by changing its container, confidence, provenance, visualization, or derived representation.

---

## UH-SB-001.2 — Authority vs. Storage Separation

**GitHub** is the authoritative machine-readable canonical *state store*.  
**Promotion Authority** is the only authority capable of changing canonical eligibility.

Registry presence in GitHub does not itself confer operational or promotion authority. GitHub holds the state; it does not autonomously promote claims.

---

## UH-SB-001.3 — Authoritative Source and Projection Sources

- **Authoritative Source:** GitHub-hosted version-controlled JSON registries and schemas (the canonical state store).
- **Projection / Authoring Sources:** Notion and any other research or working surfaces. These are strictly non-authoritative. Their content may be ingested only as candidate material under fail-closed rules.

---

## UH-SB-001.4 — Allowed Directions of Synchronization

- Notion (or other research surfaces) → SystemConnector → GitHub: candidate intake only, subject to full provenance, status, and authority checks.
- GitHub → SystemConnector → Universal Hub: read-only projection of already-authorized state.
- Bidirectional writes or Hub-originated writes that alter status, eligibility, or authority are forbidden.

---

## UH-SB-001.5 — Claim-Status Transitions

Permitted forward transitions (explicit Promotion Authority action required):

```
UNVERIFIED
    ↓
PROVISIONAL
    ↓
ELIGIBLE
    ↓
CANONICAL
```

**Containment transition (from any state):**

```
ANY STATE → QUARANTINED
```

**Quarantine Exit Rule:**  
No transition out of QUARANTINED is implicit or automatic. Release requires an explicit authorized decision by Promotion Authority and a new audit event.

No status transition may be triggered by display, synchronization, summarization, immersion, derivation, or any other non-authorized path.

---

## UH-SB-001.6 — Separation of Dimensions

Evidence quality, epistemic status, jurisdiction, and canonical eligibility are distinct dimensions. They must never be collapsed into a single scale.

A claim object must eventually distinguish at least:

- `epistemic_status`
- `evidence_state`
- `jurisdiction`
- `canonical_eligibility`

**Consequence:**  
Strong evidence does not automatically equal canonical eligibility.  
A fictional / worldbuilding statement may be fully intentional without being “false.”  
A historical correspondence may remain research evidence or provisional interpretation without becoming Terra-Gaia canon.

---

## UH-SB-001.7 — Jurisdiction Gate

A claim may be canonical only within an explicitly defined jurisdiction.  
Evidence sufficient to establish a claim in one jurisdiction does not automatically establish authority in another.

Cross-domain material (historical, literary, demonological, scriptural, or other external research) may enter the research or worldbuilding surface without promotion into Terra-Gaia canon merely because it fits a pattern.

---

## UH-SB-001.8 — Provenance Requirements

Every claim that crosses any boundary must carry, at minimum:

`claim → source → evidence → confidence → jurisdiction → status → canonical_eligibility → audit_id`

Missing any required field results in automatic rejection or quarantine.

---

## UH-SB-001.9 — Promotion Authority

Only explicitly designated actors recorded in the Promotion Authority Registry may perform a promotion event.

Every promotion event must record:

`actor → authority_role → authorization_basis → decision → audit_event`

No anonymous “system actor,” automatic process, majority vote, model output, or inferred authority is sufficient.

UH-SB-001 depends on a Promotion Authority Registry. That registry is a required dependency before any promotion path can be considered operational.

---

## UH-SB-001.10 — Rejection / Quarantine Behavior

Fail-closed is the default. The following conditions all prevent promotion and prevent authorized projection into any surface that requires a stronger epistemic status:

- INVALID
- MISSING
- STALE
- CONFLICTING
- UNVERIFIED
- absence of required provenance, authority, or evidence

Quarantined claims remain visible only to authorized researchers and are excluded from all public, immersive, and canonical surfaces.

---

## UH-SB-001.11 — Immutable Audit Record

Every boundary crossing, status change, rejection, quarantine, or release action must produce an audit event containing at least:

- timestamp
- actor
- prior state
- new state
- justification
- governing rule reference
- `integrity_reference` (version-control or cryptographic marker of the record)
- `authorization_reference` (identifier of the actual promotion decision)

Git commit hash alone is evidence of repository history; it is not necessarily sufficient evidence of who authorized a semantic state transition. Both integrity and authorization references are required.

---

## UH-SB-001.12 — Boundary Crossing Definition

A boundary crossing occurs when a claim’s authority context, visibility class, status, or provenance context changes, whether or not the underlying bytes move.

A logical projection, derived representation, or visibility-state change constitutes a boundary event even when no underlying record is physically transferred.

This definition closes the loophole: “Nothing crossed the boundary; we merely rendered it differently.”

---

## UH-SB-001.13 — Structural Read-Only Constraint on the Hub

The Universal Hub MUST NOT possess credentials, interfaces, workflows, or executable paths capable of modifying canonical state.

“We do not intend to write” is insufficient. The capability itself must be absent or structurally prohibited.

This aligns with the existing constraint that SystemConnector functions as projection/telemetry only with respect to the Hub.

---

## UH-SB-001.14 — Non-Inference of Authority

No system may infer canonical eligibility from:

- source reputation
- confidence score
- frequency of occurrence
- corroboration count
- user engagement
- majority agreement
- model output
- prior publication
- presence in GitHub
- presence in Notion
- appearance in an existing immersive surface

or any similar heuristic.

Canonical eligibility is conferred only by an explicit, audited Promotion Authority decision.

---

## UH-SB-001.15 — Fail-Closed Behavior (Summary)

If evidence, provenance, authority, required status, or any of the conditions listed in UH-SB-001.10 is missing, invalid, stale, conflicting, or unverifiable at the moment of any operation, the operation is refused and the claim remains in its prior (or quarantined) state.

No silent defaults. No optimistic promotion. No best-effort elevation.

---

## Implementation Test (Strongest Form)

If the authoritative source says “not canonical,” can any downstream path cause the Universal Hub to represent it as canonical without a separately authorized promotion event?

The correct answer must be **no** — including through transformation, inference, aggregation, caching, synchronization, derived representation, or AI-generated narration.

---

## Dependencies (Required Before Operational Use)

UH-SB-001 cannot become operational without:

1. Promotion Authority Registry
2. Claim Schema (distinguishing epistemic_status, evidence_state, jurisdiction, canonical_eligibility)
3. Audit Event Schema (with both integrity_reference and authorization_reference)
4. Evidence / Provenance Schema

---

## Current State Declaration

**STATUS:** DRAFT  
**IMPLEMENTATION:** NOT_STARTED  
**CANON PROMOTION:** BLOCKED  
**IMMERSIVE SURFACE:** BLOCKED  

This document will not move to RATIFICATION_PENDING until the amendments listed above have been reviewed and accepted, and the structural dependencies are acknowledged.

Commitment of this file establishes a proposed governance artifact only. It does not constitute ratification. Ratification remains a separate decision that must follow independent verification against the system of record.

---

*End of UH-SB-001*

**Related:** UH-FND-001 (Project Foundation) depends on this boundary.  
**Next:** After acceptance of this draft, move to RATIFICATION_PENDING only by explicit decision; then proceed to UH-EST-001 — Epistemic Status Taxonomy.
