# UH-PAR-001 — Promotion Authority Registry

**Artifact ID:** UH-PAR-001  
**Title:** Universal Hub Promotion Authority Registry  
**Format:** Markdown + JSON contract  
**Location:** terra-gaia-framework repository  
**State:** DRAFT  
**Promotion Activation:** BLOCKED  
**Depends On:** UH-SB-001 (Synchronization Boundary)  
**Schema Version:** 0.1.0  

---

## Purpose

UH-PAR-001 defines the *contract* for promotion authority.  
It declares intended authority structure without exercising, activating, or populating any authority actor.

**Critical distinction:**  
A declared control is not an enforced control until its enforcement surface is observable and independently verified.

The existence of this registry does **not** authorize any promotion event.

---

## Governing Invariant

**UH-PAR-001.1 — Authority Non-Self-Expansion**  
No actor may use authority granted by the registry to create, enlarge, or redefine its own authority.

---

## Lifecycle of the Registry Itself

```
DRAFT
  ↓
VERIFIED
  ↓
RATIFIED
  ↓
ACTIVATED
  ↓
REVOKED / SUPERSEDED
```

Current position: **DRAFT**  
`ACTIVATED = false` until the actual authority mechanism and enforcement surface have been independently verified.

No shortcut of the form “The registry says X can promote, therefore X can promote” is permitted.

---

## Registry Structure (Contract)

```json
{
  "artifact_id": "UH-PAR-001",
  "title": "Universal Hub Promotion Authority Registry",
  "schema_version": "0.1.0",
  "status": "DRAFT",
  "promotion_activation": "BLOCKED",
  "entries": [
    {
      "actor_id": "",
      "actor_type": "HUMAN|SYSTEM",
      "role": "",
      "authority_scope": [],
      "permitted_transitions": [],
      "authorization_method": "",
      "effective_from": "",
      "effective_until": null,
      "constraints": [],
      "status": "PROPOSED"
    }
  ]
}
```

**No actor identities are populated at this stage.**

---

## Required Semantic Constraints

### 1. Identity
Every authority must have a unique `actor_id`.  
Absence of a unique identity = not authorized.

### 2. Role
Identity alone does not confer authority.  
The actor must possess a defined authority role recorded in the registry.

### 3. Scope
Authority must be bounded. Scope may include (non-exhaustive):
- claim domain
- jurisdiction
- transition type
- artifact class

Unbounded or wildcard scope is forbidden.

### 4. Explicit Transitions
An actor may only execute transitions explicitly listed in its registry entry.

Examples of permitted listings:
- `UNVERIFIED → PROVISIONAL`
- `PROVISIONAL → ELIGIBLE`
- `ELIGIBLE → CANONICAL`
- `ANY → QUARANTINED`

There must be **no** wildcard of the form `"can_promote": true`.

### 5. Temporal Validity
Authority must carry an effective period (`effective_from` / `effective_until`).  
Revoked or expired authority cannot authorize a later event.

### 6. Authorization Method
The registry must specify how an authorized decision is authenticated — not merely who supposedly made it.  
`authorization_method` is a required field.

### 7. Separation of Duties
Where appropriate, the registry must be capable of requiring:

`proposer ≠ verifier ≠ ratifier`

A single actor is not assumed to be able to perform every function.

### 8. No Self-Expanding Authority
Covered by UH-PAR-001.1.  
An authority actor cannot modify its own scope or grant itself additional authority through a promotion event.

### 9. No Implicit Authority
Absence from the registry means **NOT AUTHORIZED**.  
It does not mean “probably authorized” or “default authorized.”

### 10. Audit Linkage
Every exercised authority must produce the `authorization_reference` required by UH-SB-001.  
The registry entry is a declaration of intent; the audit event is the evidence of exercise.

---

## Relationship to UH-SB-001

UH-SB-001 requires a Promotion Authority Registry before any promotion path can be considered operational.

UH-PAR-001 satisfies the *existence* of that dependency as a contract.  
It does **not** satisfy activation or enforcement.

Promotion remains BLOCKED until:

1. UH-PAR-001 reaches RATIFIED
2. Enforcement surface is designed and independently verified
3. Activation is explicitly performed and verified
4. Only then may any actor listed in an ACTIVATED registry entry exercise authority

---

## Current State Declaration

**STATUS:** DRAFT  
**PROMOTION ACTIVATION:** BLOCKED  
**ACTORS POPULATED:** None  
**ENFORCEMENT SURFACE:** Not yet identified or verified  

This artifact establishes the registry *contract* only.  
Commitment does not equal ratification, activation, or authorization of any promotion event.

---

*End of UH-PAR-001*

**Next gates (in order):**  
1. Review and acceptance of this contract  
2. Schema + enforcement design  
3. Independent verification of enforcement surface  
4. Ratification of UH-PAR-001  
5. Activation verification  
6. Only then: promotion mechanism operational
