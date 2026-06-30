# Terra-Gaia Architecture

**Version:** 1.0  
**Status:** Ratified  
**Last Modified:** 2026-06-30  

---

## Preamble

Terra-Gaia is a **canon operating system**. Its purpose is to maintain a single, immutable, deterministic source of truth for a fictional universe, while allowing projections into mutable tools (Notion, GitHub, Starmap, etc.) without contamination. This document defines the non-negotiable invariants that govern the system.

---

## 1. Single Source of Truth

The canon lives exclusively as **immutable JSON files** in the `terra-gaia-framework` GitHub repository.

- **No other system** (Notion, database, web app) holds canonical data.
- The repository is the *single source of truth* for all Nodes, Connectors, and their relationships.
- Any change to canon must be introduced via a Pull Request and pass the ratification gate.

---

## 2. The Bridge: SystemConnector

**SystemConnector** is the *only* allowed mechanism for linking a canonical Node to an external platform.

- **Forbidden:** `systems[]` arrays on Nodes, direct URL fields, or any embedded external references.
- **Rule:** Every external projection (Notion page, GitHub file, Lovable component, artwork asset, analytics dashboard, etc.) must be represented as a SystemConnector record.
- SystemConnectors carry operational metadata: `sync_status`, `last_sync`, `branch`, `external_id`, etc. This metadata is **operational log**, not canon.

---

## 3. Immutability Layers

| Layer | Content | Mutability | Storage |
|-------|---------|------------|---------|
| **Node** | Core entity (library entry, constellation, artifact, external_reference) | Immutable | Canon JSON |
| **Connector** | Canonical relationship (reveals, located_in, requires, etc.) | Immutable | Canon JSON |
| **SystemConnector** | Projection link to external platform | Mutable (status, timestamps) | Operational JSON (separate file) |
| **Traversal** | Runtime evaluation function `(Connector, Actor, State) → Result` | Ephemeral | Never persisted |
| **Witness Record** | User-specific journey audit | Ephemeral (unless logged separately) | Optional, out-of-scope |

---

## 4. Ratification Gate

Every canonical Node and Connector must carry a `ratification_status` field.

- **Values:** `"provisional"` or `"ratified"`.
- **Gate:** A Pull Request to `main` that modifies any Node or Connector **must** have all items set to `"ratified"`. If any are `"provisional"`, the PR is blocked.
- **Phase 1 (solo author):** The gate still applies. It forces a conscious declaration of intent. When you change a node, you must explicitly set it to `"ratified"` in a separate commit before merging.
- **Phase 2 (council):** The gate remains; only authorized stewards can flip the status.

---

## 5. The Three Flags (Absolute Constraints)

1. **No `systems[]` on Node.** Ever. All external mappings reside in SystemConnector.
2. **Traversal and SystemConnector are excluded from the Living Concordance.** They are not ratifiable; they do not constitute canon. Git history of SystemConnector files is operational telemetry, not ratification.
3. **ExternalReference Nodes are always non-canonical** (`canonical: false`) and require **no ratification gate**. They are citations, not governing entities.

---

## 6. The Four Ratification Items (Verbatim)

1. SystemConnector is the sole external-mapping mechanism.
2. Traversal and SystemConnector are not ratifiable.
3. `relationship_type` enum for SystemConnector includes:  
   `implemented_by`, `documented_by`, `rendered_by`, `illustrated_by`, `analyzed_by`, `localized_by`, `published_by`.
4. ExternalReference Nodes have no ratification gate.

---

## 7. Operational Logs

- SystemConnector `sync_status`, `last_sync`, and any other runtime state are **not canon**.
- They may be committed to the repository for visibility, but they are **operational telemetry**.
- The git history of these files is a *log of sync events*, not a Witness Record of canon ratification.

---

## 8. The Core Heuristic

> **“If state can be moved to a projection layer, it must be.”**

- Immutability is the default.
- Runtime concerns belong in Traversal or SystemConnector.
- Canon is eternal; projections are fluid.

---

## 9. Enforcement

- The GitHub Action (`sync-canon.yml`) validates all JSON against the schemas and enforces the ratification gate on every PR.
- It also runs the sync engine (pull from Notion, push to GitHub, update SystemConnectors).
- Compliance with these invariants is checked automatically; violations are rejected.

---

*This document is the constitution. It may only be amended by a supermajority of the council. Until council exists, the founder is the sole steward.*
