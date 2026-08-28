# UH-FND-001 — Universal Hub — Project Foundation

**Artifact ID:** UH-FND-001  
**Title:** Universal Hub — Project Foundation  
**Format:** Markdown  
**Location:** terra-gaia-framework repository  
**State:** DRAFT / RATIFICATION PENDING  
**Canon Promotion:** BLOCKED  
**Experience Implementation:** BLOCKED  
**Depends On:** UH-SB-001 (Synchronization Boundary)  

---

## Opening Invariant

The Universal Hub is a projection layer, not an authority layer. It may display, visualize, contextualize, and facilitate exploration of authorized state, but it cannot create, modify, infer, or promote canonical authority through presentation or synchronization.

---

## 1. Purpose

The Universal Hub exists to provide an immersive, evidence-aware worldbuilding environment for the Terra-Gaia universe.

Its purpose is to:

- Reveal, visualize, contextualize, and enable exploration of authorized state drawn from the Terra-Gaia canonical architecture.
- Keep unverified public-surface material, provisional hypotheses, and intentionally constructed worldbuilding visibly separated from established canon.
- Make epistemic uncertainty itself visible and navigable.
- Serve as a read-only projection/experience surface that never substitutes for the underlying governance system.

The Hub is an interface and exploratory instrument. It is not the canon, not a second source of truth, and not a promotion mechanism.

---

## 2. Scope

**In Scope**

- Projection of authorized (ratified or explicitly eligible) state from the GitHub JSON authority.
- Visualization and contextualization of claims that carry full provenance.
- Clear rendering of epistemic status (Established, Provisional, Unverified, Fictional/Worldbuilding, Quarantined).
- Facilitation of research and exploration surfaces that remain subordinate to the synchronization boundary.

**Out of Scope (until explicitly authorized by later ratified artifacts)**

- Any immersive UI or interactive experience layer.
- Automatic or silent elevation of claim status.
- Writing back into the canonical JSON authority.
- Acting as an alternate authority or parallel registry.

---

## 3. Authority Model

Authority resides exclusively in the Terra-Gaia canonical architecture:

- **GitHub JSON** = sole authoritative source of truth for Nodes, Connectors, and canonical state.
- **Notion** = authoring / working / research surface only.
- **SystemConnector** = the only permitted bridge between external surfaces and the canonical repository.
- **Universal Hub** = pure consumer of projected, already-authorized state.

No component of the Universal Hub may acquire, infer, inherit, or manufacture authority over any claim.

---

## 4. System Boundaries

| Boundary | Role | Authority |
|----------|------|-----------|
| GitHub (terra-gaia-framework) | Canonical JSON authority | Authoritative |
| Notion | Authoring / research surface | Non-authoritative |
| SystemConnector | Controlled, auditable bridge | Operational only |
| Universal Hub | Read-only projection / experience surface | Non-authoritative |

Any movement of claims across these boundaries is a synchronization event and is governed by UH-SB-001. A synchronization event is never a canon-promotion event.

---

## 5. Surface Definitions

- **Canonical Surface** — GitHub-hosted version-controlled JSON registries and schemas. Only material that has passed the ratification gate may reside here as established canon.
- **Authoring Surface** — Notion (and any future research workspaces). Material here is provisional by default.
- **Projection Surface** — The Universal Hub. It consumes and renders projected state. It does not originate or elevate authority.
- **Quarantine Surface** — Explicitly excluded material that remains visible only to authorized researchers and is barred from public or immersive propagation.

---

## 6. Epistemic Separation

The Hub must keep the following categories visibly distinct at all times:

- **Established** — Evidenced and eligible for canonical treatment (or already ratified).
- **Provisional** — Supported hypothesis or synthesis awaiting verification.
- **Unverified** — Interesting material with insufficient evidence.
- **Fictional / Worldbuilding** — Intentionally constructed material.
- **Quarantined** — Explicitly excluded from public or canonical propagation.

Epistemic status is never altered by display, visualization, narration, aggregation, or user engagement within the Hub.

---

## 7. Synchronization Principle

All synchronization is unidirectional with respect to authority:

- Candidate material may flow from Notion → SystemConnector → GitHub only under the rules of UH-SB-001 (fail-closed, provenance-required).
- Authorized state may flow from GitHub → SystemConnector → Universal Hub as read-only projection.
- The Hub itself initiates no writes that alter status, eligibility, or authority.

This principle is subordinate to and dependent upon UH-SB-001.

---

## 8. Non-Authority of the Hub

**UH-FND-001.8 — Hub Non-Authority Rule**

No Universal Hub component, projection, interface, agent, visualization, narration, aggregation, transformation, synchronization event, or user engagement may:

- Create canonical authority,
- Modify existing authority,
- Infer authority from presentation,
- Promote epistemic status, or
- Substitute itself for the GitHub JSON authority or the SystemConnector.

The Hub reveals authorized state; it does not authorize state.

---

## 9. Dependencies on UH-SB-001

This Foundation Document is incomplete and non-operative without UH-SB-001 (Universal Hub Synchronization Boundary).

UH-SB-001 defines:

- Authoritative source and projection sources
- Allowed synchronization directions
- Claim-status transitions
- Provenance requirements
- Promotion authority
- Rejection / quarantine behavior
- Immutable audit record
- Boundary-crossing criteria
- Fail-closed behavior

Until UH-SB-001 is itself ratified, no synchronization into or out of the Universal Hub is authorized, and this Foundation remains blocked from implementation effects.

---

## 10. Explicit Implementation Gate

**No Experience Layer implementation is authorized until UH-FND-001 and UH-SB-001 have been independently verified and ratified.**

Until both artifacts reach ratified status:

- Canon promotion remains BLOCKED.
- Experience / immersive surface implementation remains BLOCKED.
- The Hub may not be treated as operational.

This document, by being committed in DRAFT / RATIFICATION PENDING state, establishes a proposed governance artifact only. Commitment does not equal ratification. Ratification is a separate, explicit decision that must follow independent verification against the system of record.

---

*End of UH-FND-001*

**Next planned artifact:** UH-EST-001 — Epistemic Status Taxonomy  
**Governing sequence:** Synchronization Boundary (UH-SB-001) → Project Foundation (UH-FND-001) → Epistemic Status Taxonomy → Core Scope → Evidence Layer → Experience Layer
