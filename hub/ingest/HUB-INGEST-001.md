ID: HUB-INGEST-001
Version: 1.0
Status: PROPOSED — NOT RATIFIED
Author: <your-name-or-handle>
Date: 2026-08-29
Repository: nashondas-oss/terra-gaia-framework
Anchor-commit: cc70eed7c5f98991a7c9e9237c6f996622a0f0ef
Proposal-path: hub/ingest/HUB-INGEST-001.md

Purpose
- Define the Hub read‑only ingest gate that produces an immutable diagnostic and a snapshot artifact (SNAPSHOT EMISSION / PROJECTION ELIGIBILITY) without any mutative side effects or automatic proposals.
- This is a PROPOSAL only. It neither ratifies nor authorizes canonical mutation or importer activation.

Core invariant (explicit)
- HUB-INGEST-001 does NOT determine authority. It only determines whether a source snapshot satisfies the current frozen retrieval, integrity, schema, and provenance prerequisites for downstream governance consideration. Valid snapshot ≠ authorized projection.

Scope
- Inputs: owner/repo and ref (branch or commit SHA) and an ingest manifest identifier/version that declares the ordered set of paths to include.
- Outputs: immutable diagnostic JSON (HUB-DIAG-001) and an immutable snapshot artifact stored in an artifact store (no writes into canonical repo).
- Exclusions: no corpus scoring, no canonical mutation, no automatic PRs, no importer activation, no changes to main.

Principles / invariants
1. Authentication increases access only; it never confers authority.
2. All decisions about who is authorized (allowlist) are made against a governed policy artifact (named by allowlist_source + allowlist_version) — the ingest runner must reference that artifact for policy evaluation.
3. A failed validation produces no projection and causes the gate to stop. Retrieval failure implies validation is not run.
4. Deterministic payload selection + deterministic canonicalization + a frozen fingerprint contract are all required for snapshot reproducibility. None may be left underspecified.
5. Diagnostics and snapshots are write‑once immutable artifacts. Subsequent runs produce distinct diagnostics.

Deterministic payload selection (manifest authoritative)
- The ingest manifest (identified by manifest_id and manifest_version) is authoritative for payload selection. It MUST be provided to the ingest runner and MUST list an ordered array of normalized repository paths to include in the snapshot.
- The manifest is itself versioned and identified in diagnostics (manifest_id + manifest_version).
- canonical_payload is the canonical serialization of the manifest‑defined ordered set (see Fingerprint Contract).

High-level read-only flow
1. AUTHENTICATED REQUEST — read‑scoped token only.
2. AUTHORIZED REPOSITORY — confirm owner/repo/ref reachable.
3. READ-ONLY PAYLOAD — fetch the manifest‑listed paths at commit_sha. No writes, no refs, no PRs.
4. SOURCE IDENTITY VALIDATION — verify fetched commit_sha equals requested ref; verify commit reachability.
5. SCHEMA VALIDATION — validate each canonical JSON file against authoritative schemas (AJV or equivalent).
6. PROVENANCE VALIDATION — evaluate commit signature presence/validity and evaluate author per governed allowlist policy artifact. Preserve both policy reference and evidence in the diagnostic.
7. SNAPSHOT EMISSION / PROJECTION ELIGIBILITY — canonicalize manifest‑defined payload, compute snapshot_id per the canonical fingerprint contract, emit snapshot artifact and diagnostic JSON to immutable artifact store.
8. STOP — no mutative action; projection and proposals are governed by HUB-EVT-001.

Failure handling rules (stronger)
- If retrieval_status == failure then validation_status MUST == not_run. Diagnostic must include retrieval_error.
- If retrieval_status == success then commit_sha MUST be reachable and the diagnostic MUST include fingerprint and snapshot_id.
- If retrieval succeeded but validation failed, fingerprint and snapshot_id exist (because payload was retrieved) and validation_errors enumerate failures.

Deterministic fingerprint contract (frozen; byte-level)
- Canonicalization standard: RFC 8785 (JSON Canonicalization Scheme, JCS). Implementations MUST reference the canonical implementation/version used. The diagnostic fingerprint canonicalization block must include canonicalization.name = "RFC8785-JCS" and canonicalization.version = "<implementation-version-or-spec-vX>".
- Payload boundary and serialization:
  - The ingest manifest defines an ordered array of normalized paths: [p1, p2, ..., pn].
  - For each path pi (in manifest order), the ingest runner constructs a file record:
      record_i = UTF-8(normalize_path(pi)) || 0x1F || UTF-8(JCS(canonicalize_content_of(pi)))
      where 0x1F is the ASCII Unit Separator and JCS() yields the RFC8785 canonical JSON text encoded to UTF‑8 bytes for JSON files; for non-JSON files, use raw UTF‑8 bytes.
  - canonical_payload = record_1 || 0x00 || record_2 || 0x00 || ... || 0x00 || record_n
    (records separated by a single NULL byte 0x00).
- Snapshot hash computation:
  - Let commit_sha_bytes = UTF-8(lowercase 40-char commit SHA).
  - Let manifest_id_version = UTF-8(manifest_id + "@" + manifest_version).
  - Let hash_input = canonical_payload || 0x00 || commit_sha_bytes || 0x00 || manifest_id_version.
  - snapshot_id = SHA-256(hash_input) encoded as 64‑hex lowercase string.
- Contract: snapshot_id MUST equal fingerprint.value in the diagnostic. Implementations MUST assert equality before emitting diagnostics.

Provenance policy vs evidence
- Diagnostic must contain:
  - provenance_checks.allowlist_source and allowlist_version (the policy artifact referenced).
  - provenance_checks.evidence: concrete facts evaluated — commit_signature { required, present, valid }, commit_author, commit_timestamp, and policy_decision { decision: "allow"|"deny", rationale: string }.
- Implementations must not record only the decision; they must preserve the evidence used to reach it.

Immutability & audit
- Diagnostics and snapshots are write‑once. Store in an immutable artifact location (CI artifact with retention policy or dedicated audit store). The diagnostic must include retrieval_timestamp and an immutable artifact URL to the snapshot.

Operational notes
- The ingest runner MUST NOT create PRs, refs, or open advisory proposals — those actions belong to HUB-EVT-001.
- The ingest runner MUST NOT open PRs or create refs as part of this gate.
- The ingest runner MUST include canonicalization identifier/version in fingerprint.canonicalization so independent verifiers can reproduce the payload.

Acceptance criteria for the proposal
- Ingest runner test executes read-only retrieval for success and failure cases and produces diagnostics matching the schema and invariants above.
- Diagnostics clearly differentiate retrieval vs validation vs provenance failures and include snapshot_id when appropriate.
- No mutative side-effects occur during testing.

References
- RFC 8785 — JSON Canonicalization Scheme (JCS)
- hub/diag/HUB-DIAG-001.schema.json (revised)

Status: PROPOSED — revision required until governance ratifies under HUB-EVT-001
