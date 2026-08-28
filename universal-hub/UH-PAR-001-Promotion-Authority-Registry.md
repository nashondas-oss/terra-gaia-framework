# UH-PAR-001: Universal Hub Promotion Authority Registry

**Status:** DRAFT  
**Promotion Activation:** BLOCKED  
**Schema Version:** 0.1.0  
**Artifact ID:** UH-PAR-001

---

## Overview

This registry defines the structure, constraints, and lifecycle for authority actors authorized to perform claim promotion within the Universal Hub ecosystem. It establishes **what authority looks like** without yet naming or activating any authority actor.

The registry serves as the contract between:
- **UH-SB-001** (Synchronization Boundary) — the rules that govern promotion
- **Enforcement mechanism** (to be verified separately) — the system that prevents unauthorized transitions
- **Authority actors** (to be registered later) — entities that may exercise promotion authority

**Critical distinction:** Registry existence ≠ authority activation. No actor may exercise promotion authority until:
1. UH-PAR-001 is ratified
2. The enforcement mechanism is independently verified
3. UH-PAR-001 transitions to ACTIVATED status
4. The specific actor entry is verified present and within temporal validity

---

## Registry Schema

```json
{
  "artifact_id": "UH-PAR-001",
  "title": "Universal Hub Promotion Authority Registry",
  "schema_version": "0.1.0",
  "status": "DRAFT",
  "promotion_activation": false,
  "lifecycle": {
    "current_state": "DRAFT",
    "state_sequence": ["DRAFT", "VERIFIED", "RATIFIED", "ACTIVATED", "REVOKED/SUPERSEDED"],
    "transition_requirements": {
      "DRAFT_to_VERIFIED": "schema completeness + constraint audit",
      "VERIFIED_to_RATIFIED": "explicit governance decision",
      "RATIFIED_to_ACTIVATED": "enforcement mechanism verification + independent validation",
      "ACTIVATED_to_REVOKED": "governance decision + audit trail"
    }
  },
  "entries": []
}
```

### Entry Template (Unpopulated)

Each authority entry follows this structure:

```json
{
  "entry_id": "PAR-ENTRY-[UUID]",
  "actor_id": null,
  "actor_type": "HUMAN|SYSTEM|COLLECTIVE",
  "actor_role": null,
  "authority_scope": {
    "claim_domains": [],
    "jurisdictions": [],
    "transition_types": [],
    "artifact_classes": []
  },
  "permitted_transitions": [],
  "authorization_method": null,
  "temporal_validity": {
    "effective_from": null,
    "effective_until": null,
    "revocation_date": null
  },
  "constraints": [],
  "audit_linkage": {
    "authorization_reference_format": "per UH-SB-001.7",
    "signature_requirement": null,
    "log_requirement": null
  },
  "status": "PROPOSED",
  "last_verified": null,
  "verification_evidence": null
}
```

---

## Semantic Constraints

### 1. Identity Requirement

**Constraint:** Every authority entry must have a unique `actor_id`.

- `actor_id` must be immutable once an entry transitions beyond PROPOSED status
- `actor_id` must be globally resolvable (fully qualified name, cryptographic key, or stable identifier)
- Duplicate `actor_id` values across entries are forbidden
- Absence of `actor_id` means the entry remains PROPOSED and cannot authorize any action

**Rationale:** Authority cannot be exercised on behalf of an unnamed principal. Ambiguity about identity is a boundary violation per UH-SB-001.

---

### 2. Role Requirement

**Constraint:** Identity alone does not confer authority. An actor must possess a defined authority role.

- `actor_role` must be explicitly listed (e.g., CLAIM_VERIFIER, JURISDICTION_ARBITER, CANONICAL_ARBITER)
- Roles must be defined in a separate role taxonomy (not inline)
- An actor may hold multiple roles, but each role is listed separately
- Role changes require a new registry entry, not modification of an existing entry

**Rationale:** Authority is role-based. A person or system is authorized for specific functions, not for "everything they claim to need."

---

### 3. Scope Boundedness

**Constraint:** Authority must be bounded across multiple dimensions:

- **claim_domains**: List of claim categories this actor can promote (e.g., ["cosmological", "frequency-protocol"], not "*")
- **jurisdictions**: Geographical, organizational, or logical boundaries (e.g., ["terra-gaia-core"], not "all")
- **transition_types**: Specific state transitions allowed (see Constraint 4)
- **artifact_classes**: Types of artifacts this authority applies to (e.g., ["claim", "evidence"], not "*")

**Rationale:** Unbounded authority defeats the purpose of a registry. UH-SB-001 requires jurisdiction gating. This constraint enforces it.

---

### 4. Explicit Transitions

**Constraint:** An actor may only execute transitions explicitly listed in `permitted_transitions`.

Example permitted transitions:
```json
"permitted_transitions": [
  "UNVERIFIED → PROVISIONAL",
  "PROVISIONAL → ELIGIBLE",
  "ELIGIBLE → CANONICAL"
]
```

**Forbidden patterns:**
- Wildcard transitions: `"ANY → CANONICAL"` (not allowed)
- Implicit authority: `"can_promote": true` (not allowed)
- Reverse transitions: `"CANONICAL → PROVISIONAL"` (unless explicitly authorized for specific use case)
- Quarantine bypass: `"ANY → ELIGIBLE"` (quarantine requires explicit unquarantine authority)

**Rationale:** Explicit transitions make authorization auditable and prevent scope creep. An actor cannot accidentally or deliberately move claims through unauthorized state paths.

---

### 5. Temporal Validity

**Constraint:** Authority must have a defined effective period:

```json
"temporal_validity": {
  "effective_from": "ISO-8601 timestamp",
  "effective_until": "ISO-8601 timestamp or null for indefinite",
  "revocation_date": "ISO-8601 timestamp if revoked"
}
```

- If `revocation_date` is set, the entry is immediately inactive regardless of `effective_until`
- An authorization event cannot be validated if it occurred before `effective_from` or after `effective_until` (or after `revocation_date`)
- Expired authority cannot authorize later events retroactively

**Rationale:** Authority can be revoked, delegated, or limited in duration. The enforcement mechanism must verify not only that the actor exists but that the actor's authority was valid at the time the decision was made.

---

### 6. Authorization Method

**Constraint:** The registry must specify how an authorized decision is authenticated.

Example methods:
- **cryptographic_signature**: Actor signs authorization with private key; signature verified against public key in registry
- **mfa_token**: Actor provides multi-factor authentication token issued by trusted provider
- **governance_decision**: Actor's authority requires a separate governance committee decision (not unilateral)
- **consensus_threshold**: Actor's authority requires N of M collective agreement
- **temporal_audit**: Actor's identity verified against audit log at time of decision

**Rationale:** A declared authority is not an enforced authority until the enforcement mechanism can verify that the decision was actually made by the authorized actor. "The registry says so" is insufficient.

---

### 7. Separation of Duties

**Constraint:** Where appropriate, the registry must be capable of expressing required role separation:

Example:
```json
"constraints": [
  {
    "type": "separation_of_duties",
    "description": "proposer ≠ verifier ≠ ratifier",
    "prohibited_combinations": [
      ["CLAIM_PROPOSER", "CLAIM_VERIFIER"],
      ["CLAIM_VERIFIER", "CANONICAL_ARBITER"]
    ]
  }
]
```

- If an actor holds multiple roles, the registry must explicitly state whether certain role combinations are forbidden
- Separation of duties can be enforced at the registry level or at the enforcement mechanism level (must be specified)

**Rationale:** Critical decisions (especially promotion to canonical status) require independent verification. One actor should not be able to propose, verify, and ratify their own claim.

---

### 8. Authority Non-Self-Expansion (UH-PAR-001.1)

**Invariant:** No actor may use authority granted by the registry to create, enlarge, or redefine its own authority.

**Forbidden actions:**
- An actor with CLAIM_VERIFIER role cannot modify UH-PAR-001 to grant itself CANONICAL_ARBITER role
- An actor with authority over domain "cosmological" cannot use a promotion event to expand its scope to "all-domains"
- An actor cannot create a new registry entry for itself
- An actor cannot vote on the ratification of UH-PAR-001 if that ratification would activate their own authority

**Constraint in registry:**
```json
"constraints": [
  {
    "type": "non_self_expansion",
    "description": "UH-PAR-001.1: No actor may expand its own authority",
    "enforcement_level": "MANDATORY",
    "verification": "every authority decision must be audited to confirm the actor did not modify UH-PAR-001 or create a new entry for themselves"
  }
]
```

**Rationale:** This is the primary guard against authority creep. Without it, an actor granted limited authority could unilaterally expand it through self-referential decisions.

---

### 9. No Implicit Authority

**Constraint:** Absence from the registry means NOT AUTHORIZED.

- If an `actor_id` is not present in the registry, or if present but expired/revoked, that actor cannot authorize any promotion
- There is no fallback, no "probably authorized," no "parent authority grants it"
- An authorization event must reference a specific, current, non-revoked registry entry

**Rationale:** Implicit authority is a boundary violation. This is the flip side of UH-PAR-001.1: the system assumes no authority unless explicitly declared and verified.

---

### 10. Audit Linkage

**Constraint:** Every exercised authority must produce an authorization reference conforming to UH-SB-001.7.

```json
"audit_linkage": {
  "authorization_reference_format": "per UH-SB-001.7",
  "signature_requirement": "actor must cryptographically sign or otherwise authenticate the decision",
  "log_requirement": "every authorization event must be logged with: [timestamp, actor_id, decision, authorization_method, evidence]",
  "evidence_format": "per UH-EVIDENCE-001 (when defined)"
}
```

- Authorization references must be immutable once recorded
- Authorization references must link back to the registry entry that authorized the decision
- If enforcement mechanism or evidence schema cannot produce the required audit trail, the authorization is void

**Rationale:** Authority without audit is unverifiable. UH-SB-001 explicitly requires authorization_reference. This constraint makes that requirement operational in the registry.

---

## Lifecycle States

### DRAFT
- Schema and constraints are under review
- No actor may exercise authority claimed under this registry
- Entries are PROPOSED and serve as documentation only
- Eligible for modification without full governance process

### VERIFIED
- Schema completeness has been audited
- Constraints have been reviewed for semantic coherence
- Entries are internally consistent
- Transitions to RATIFIED require explicit governance decision
- Still no authority activation

### RATIFIED
- Explicit governance decision has adopted this registry as binding
- Registry constraints are now governing rules
- **But authority is not yet operational**
- Enforcement mechanism verification is still required before ACTIVATED

### ACTIVATED
- Enforcement mechanism has been independently verified to prevent unauthorized transitions
- Authority actors listed in the registry can now exercise their designated authority
- All UH-SB-001 constraints are verified as enforced
- Promotion mechanism is operational

### REVOKED / SUPERSEDED
- Registry is no longer authoritative
- New registry version or alternative governance replaces this one
- Authority actors lose authority upon transition to this state
- Historical record is preserved for audit purposes

---

## Current Status

**Registry status:** DRAFT  
**Promotion activation:** BLOCKED  
**Actor entries:** 0 (unpopulated)  
**Enforcement mechanism:** NOT YET VERIFIED  
**Next action:** Schema verification audit before proceeding to VERIFIED status

---

## Blocked Actions

Until this registry transitions to ACTIVATED:

- ❌ No actor may promote claims, regardless of registry entries
- ❌ UH-PAR-001 entries cannot be used as authorization for any decision
- ❌ Claim promotion mechanism remains inert
- ❌ UH-EST-001 (Experience Surface) remains blocked
- ❌ Canon promotion authority remains blocked

---

## Dependencies

- **Upstream:** UH-SB-001 (RATIFIED) — defines the synchronization boundary this registry serves
- **Parallel:** Enforcement mechanism design (to be specified)
- **Downstream:** UH-CSA-001 (Claim Schema), UH-EVIDENCE-001 (Evidence/Provenance Schema), UH-EST-001 (Experience Surface)

---

## Registry Invariants

1. **UH-PAR-001.1 — Authority Non-Self-Expansion:** No actor may use registry-granted authority to modify UH-PAR-001 or create entries for themselves
2. **UH-PAR-001.2 — No Implicit Authority:** Absence from registry = NOT AUTHORIZED
3. **UH-PAR-001.3 — Temporal Binding:** Authority validity is time-bound; expired authority cannot authorize future events
4. **UH-PAR-001.4 — Explicit Transitions:** Only transitions in permitted_transitions are authorized; wildcard authority is forbidden
5. **UH-PAR-001.5 — Audit Linkage:** Every authorization must produce UH-SB-001.7-compliant authorization reference
6. **UH-PAR-001.6 — Scope Boundedness:** Authority is bounded across identity, role, domain, jurisdiction, transition type, and artifact class
7. **UH-PAR-001.7 — Registry-Enforcement Separation:** Registry defines intended authority; enforcement mechanism independently verifies it is operational

---

## Next Steps

1. **Schema completeness review** — verify this template covers all required dimensions
2. **Semantic constraint audit** — confirm each constraint is operationally meaningful
3. **Enforcement mechanism specification** — define how the system will prevent unauthorized transitions
4. **Independent verification design** — specify verification procedure before ACTIVATED state
5. **Transition to VERIFIED status** — only after all above are complete and audited

**UH-PAR-001 remains DRAFT. Promotion authority remains BLOCKED.**