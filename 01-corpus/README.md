Handoff Protocol — provenance-first (README)

Purpose
- Define an unambiguous, provenance-oriented handoff protocol for delivering a frozen corpus and related artifacts from a creator to an execution environment and then to an independent evaluator.
- Preserve strict separation of concerns: the corpus is a frozen specification, the implementation executes against it and records raw facts, and the evaluator consumes only those facts to produce verdicts.
- Prevent any claimed identity, hash, or signature from being treated as authoritative without independent verification.

Roles
- Creator: prepares the frozen corpus and supplies a handoff bundle.
- Receiver / Executor: the party that will run the implementation against the corpus and produce raw evidence. May be the implementer or an independent runner.
- Verifier: the party that independently recomputes and confirms corpus identity prior to any execution. Verifier may be the receiver, an independent party, or both.
- Evaluator: the independent comparator that consumes evidence records and the frozen corpus to produce comparator_output. The evaluator never runs the implementation or modifies evidence.

High-level rule (non-negotiable)
- A reported hash is not accepted as authoritative until it is recomputed and matched by the Verifier.
- Execution is permitted only after an independent MATCH between the supplied manifest hash and the recomputed manifest hash.
- UNVERIFIED is not a PASS and not a FAIL; it is a state that requires remediation before proceeding to execution unless an explicit policy (documented and signed) allows exceptions.

What is handed over (exact artifact set)
- Corpus bundle (creator → receiver): A compressed archive (tar.gz or zip) or git reference containing:
  - manifest.json (corpus manifest, includes declared hash algorithm)
  - cases/ (all case JSON files)
  - fixtures/ (referenced fixture files)
  - SHA256SUMS (text file listing canonical file paths and hashes as computed by the creator; treated as a claimed inventory only)
  - README and any schema files included in the corpus
- Optional metadata: signed release note, optional creator-supplied signature(s) for integrity (signing policy must be separately configured).
- Transfer record: metadata describing transfer method, who transferred, and when (see handoff-record template below).

Who receives it
- The Receiver (execution agent) receives the bundle by one of:
  - Direct copy (scp/https/s3), or
  - Git reference / tagged commit, or
  - Signed release artifact (e.g., OCI image or GitHub release)
- The Verifier must have read access to the received bundle and must be able to compute cryptographic hashes locally.

How the corpus identity is verified (minimal, mandatory steps)
1) Receipt: Receiver stores the received artifact in an immutable location (see storage below) and records:
   - transfer_method (HTTP/HTTPS/S3/Git)
   - transfer_source (URL or origin)
   - received_at (RFC3339 timestamp)
   - receiver_identity (who pulled/saved the artifact)
   - transfer_artifact_id (file name or git ref)
2) Independent recomputation:
   - Unpack the received artifact into a clean directory.
   - Canonicalize each JSON file using the agreed canonicalization (see Canonicalization below).
   - Recompute hashes for all files listed in manifest (and for the manifest itself).
   - Compute a manifest_hash_computed = SHA-256(canonicalized manifest.json).
3) Compare:
   - Compare manifest_hash_computed to the supplied_manifest_hash (the creator-supplied value, if any).
   - Record MATCH (exact equality) or MISMATCH and store both values in the handoff record.
4) Decision:
   - If MATCH → corpus identity is VERIFIED and execution may proceed.
   - If MISMATCH → corpus identity is UNVERIFIED. Do NOT execute. Follow remediation (see below).
   - If no supplied_manifest_hash was provided → treat as UNVERIFIED until a reproducible method for identifying the corpus is established and recorded.

Canonicalization (required)
- JSON canonicalization MUST be deterministic. Recommended options (choose and document one for the project):
  - Strong: Use a JCS-compatible canonicalizer (JSON Canonicalization Scheme) for byte-for-byte determinism.
  - Practical: jq -S . (sort keys) then normalize UTF-8, strip insignificant whitespace, and ensure no platform-specific EOL differences; compute SHA-256 on the resulting bytes. Note: jq -S is acceptable only if all parties have agreed and documented that it is the canonicalization method; JCS is preferable for strictness.
- Always compute over UTF-8 encoded bytes.
- Record the canonicalization method and tool+version in the handoff record.

How hashes are independently recomputed (commands)
- Example (jq-based):
  cd /path/to/unpacked/corpus
  jq -S . manifest.json | tee /tmp/manifest.canonical.json | sha256sum
- For each case file:
  jq -S . cases/IEH-CROWN-001.json | sha256sum
- To generate SHA256SUMS (verifier side) after canonicalization:
  find cases -type f -name '*.json' -exec sh -c 'jq -S . "$1" | sha256sum | cut -d" " -f1 > "{}".sha256' _ {} \;
- Prefer JCS tool if strict canonicalization required:
  jcs-canonicalize manifest.json | sha256sum
- Record exact commands and tool versions in the handoff-record.

What the evaluator is permitted to access
- The evaluator is permitted to read:
  - The VERIFIED corpus (manifest + case files + fixtures).
  - Evidence records produced by the Receiver/Executor.
  - Transfer/handoff records and replay artifacts.
- The evaluator is NOT permitted to:
  - Modify the corpus files in place.
  - Modify evidence records (only append audit trail or produce comparator_output).
  - Execute the target implementation or request the Receiver to re-run tests on its behalf.
- Access must be read-only (file permissions, object-store policies, or separate environment) and logged.

What the evaluator is prohibited from modifying
- Corpus files in the canonical corpus store.
- Creator-supplied artifacts (fixtures, manifest).
- Evidence records (evidence must be preserved as-written).
- The evaluator may produce separate comparator_output artifacts and audit logs; these must be stored in a distinct namespace and never written back into the evidence store used for evaluation.

How the target implementation is identified (minimum assertions)
- If the target is a Git repository:
  - repository_url
  - commit_sha (full 40-char SHA) used for the execution
  - signed_tag (if used) and its signature metadata
  - build artifact checksum(s) (artifact tarball or binary, SHA-256)
- If the target is a container image:
  - image_name
  - image_digest (sha256:...)
  - registry and pull URL
- If the target is a binary/service:
  - binary name
  - artifact checksum (SHA-256)
  - build environment metadata (build command, tool versions)
- The implementation_identity recorded in each evidence record must include these values and must be recomputable from artifacts retained in the execution archive.

How execution is initiated (procedural options)
- Option A (Implementer-run):
  - Implementer runs the tests locally/CI against the verified corpus and writes evidence records and execution artifacts to an immutable evidence bundle then transfers to the evaluator.
- Option B (Independent executor):
  - An independent executor (separate identity) pulls the verified corpus and runs a documented runner to produce evidence.
- Required step before any execution:
  - The executor must create an execution-record that includes:
    - execution_id (UUID)
    - who_initiated (executor identity)
    - started_at / finished_at timestamps
    - environment_description (OS, container runtime, random seeds, CLI)
    - implementation_identity (commit/image/digest)
    - invocation_command (exact command-line)
    - artifacts produced (list with checksums)
  - The execution-record is stored in the execution bundle and its checksum recorded in the handoff record.

Where raw facts are written (evidence store)
- Evidence file naming:
  evidence-{case_id}-{timestamp}-{execution_id}.json
  Example: evidence-IEH-CROWN-001-2026-08-19T12:34:56Z-6f2b3a.json
- Each evidence file MUST conform to evidence_record.schema.json.
- Execution bundles (complete run) should be stored as:
  evidence-bundle-{execution_id}.tar.gz
  and accompanied by evidence-bundle-{execution_id}.sha256
- Evidence store requirements:
  - Append-only or immutable once written (object-store with versioning + retention policy or WORM).
  - Access controls ensuring only authorized parties can write; evaluator has read-only access.
  - Logged access, including who uploaded what and when.

How failed executions are preserved
- All failed executions are PRESERVED identically to successful ones.
- The evidence bundle MUST include:
  - stdout/stderr logs
  - exit codes
  - any stack traces
  - the exact invocation_command and environment snapshot
  - partial outputs and the location of any produced artifacts
- The evaluator will treat failure artifacts as legitimate evidence and will either produce a comparator_output (if admissible facts exist) or UNDETERMINED if required provenance is missing.

How replay is performed
- Replay is permitted only when:
  - The execution bundle contains the exact implementation artifacts (commit SHA or image digest) and the invocation_command/environment snapshot.
  - The replayer is a separate identity from the original executor when independence testing is required.
- Replayer must produce a replay-record containing steps taken, differences observed, and checksums of regenerated artifacts.
- The replay-record itself is evidence and must be stored and preserved.

How evaluator output is separated from implementation output
- Implementation outputs: stored under evidence/ or evidence-bundles/ — must contain raw_output, ledger_events, state_transition, signatures, etc.
- Evaluator outputs: comparator_output-{case_id}-{execution_id}.json
  - Stored in a different namespace: comparator_outputs/
  - Evaluator must not inject comparator_output into evidence/ or otherwise mark comparator_output as an evidence artifact.
- For traceability, comparator_output must reference evidence file hashes but must not be included among the raw evidence consumed.

How provenance of resulting evidence is established
- Every evidence record must contain:
  - input_hash (canonicalized case input SHA-256)
  - implementation_identity (commit/image/digest)
  - execution_identity (execution_id, executor identity, timestamp)
  - evidence_hash (SHA-256 of the canonicalized evidence record)
  - storage_location (URL/object path)
  - signatures (optional; signer identity and signature blob)
- The handoff-record must include:
  - supplied_manifest_hash (if provided by creator)
  - manifest_hash_computed (verifier recomputed)
  - manifest_match boolean
  - verifier_identity and timestamp
  - transfer metadata
- The comparator_output must include evidence_hashes and the exact manifest_hash_computed used in evaluation.

What constitutes an independence failure
- Independence failures examples:
  - The same cryptographic key is used to sign both the evidence and the comparator_output without separate, tested trust anchors.
  - The evaluator runs the implementation (direct or via remote API call) as part of evaluation.
  - The same identity prepared the corpus, executed the implementation, and served as sole verifier without an established independence rationale.
  - The verifier cannot recompute hashes due to missing canonicalization tools or ambiguous file encodings.
- When an independence failure is detected, record it explicitly in the handoff-record and comparator_output.audit_trail and treat related claims as UNVERIFIED for the purposes of downstream decisions.

What happens when any prerequisite cannot be verified
- If manifest_hash_computed ≠ supplied_manifest_hash (MISMATCH) or cannot be computed:
  - DO NOT EXECUTE.
  - Create a HANDOFF FAILURE record with:
    - mismatch_reason
    - remediation_options (re-transfer, notarized re-release, independent mirror)
    - timestamps and actor identities
  - Hand off the HANDOFF FAILURE record to the Creator for remediation and to any designated oversight party.
- If required provenance items specified by a case are missing in evidence:
  - Evaluator returns UNDETERMINED for that case (Decision precedence: missing/unverifiable provenance → UNDETERMINED).
  - The evidence must be preserved; the executor may be asked to rerun (but only re-execution performed and recorded by the executor; evaluator must not run the implementation).
- Document whichever remediation path is chosen and include it in the audit trail. Do not conflate unresolved verification with target failure.

UNVERIFIED ≠ FAILED — explicit policy
- Policy: Unverified corpus or evidence must be treated as an operational state requiring remediation. It must NOT be used to justify conclusions about the target implementation.
- Record the difference in any public reporting: CORPUS_VERIFICATION_STATUS: {VERIFIED | UNVERIFIED | MISMATCH}.

HANDOFF FAILURE ≠ TARGET FAILURE — explicit policy
- Policy: A handoff or verification problem is an incident about exchange and provenance. It is not evidence that the implementation failed to meet the case goals.
- Preserve both pieces of information separately: handoff incident logs and execution logs.

Handoff-record template (required fields)
- File name: handoff-record-{bundle_id}.json
{
  "bundle_id": "<uuid>",
  "transfer_method": "https|s3|git|oci",
  "transfer_source": "<URL or origin>",
  "received_at": "2026-08-19T12:34:56Z",
  "receiver_identity": {"id":"<principal>","type":"human|service"},
  "supplied_manifest_hash": "<sha256-or-empty>",
  "manifest_hash_computed": "<sha256-or-empty>",
  "manifest_match": true|false,
  "canonicalization_method": "<jcs|jq -S|...>",
  "verifier_identity": {"id":"<principal>","type":"human|service"},
  "verification_timestamp": "2026-08-19T12:35:10Z",
  "verification_commands": ["exact shell commands used"],
  "storage_location": "<object-store-path-or-git-ref>",
  "notes": "If mismatch, list remediation steps taken or pending"
}

Evidence record requirements (summary)
- Must conform to evidence_record.schema.json.
- Must include input_hash (SHA-256 of canonicalized case input).
- Must include implementation_identity and execution_identity.
- Must include a local evidence_hash computed by verifier/receiver after canonicalization.
- Storage location and uploader identity must be recorded.

Audit trail and logging
- All steps (transfer, unpack, recompute hash, decision to run, execution start/stop, evidence upload) must be logged with RFC3339 timestamps and actor identities.
- Audit trail entries should be appended-only and stored with strong integrity protections (object-store immutability or signed timestamped logs).
- Comparator_output must include the exact audit trail entries that were consulted.

Recommended tooling and practical notes
- For strong canonicalization use a JCS implementation (preferred). Document tool name and version.
- For JSON-only corpora, jq -S . is acceptable if agreed in advance, but be explicit about limitations (e.g., floating point representation).
- Use stable, reproducible builds for the implementation artifacts; store build artifacts with SHA-256 checksums.
- Use an immutable object store (S3 with object lock, GCS with retention, or equivalent) for evidence bundles.
- Require that any automated transfer use TLS and server-side integrity checks (e.g., Content-MD5 or SHA-256 via headers).
- If signatures are used, treat them as integrity/authorship metadata only. Evaluate signer independence separately before trusting signatures for provenance claims.

Checklist (pre-execution)
Creator:
- [ ] Produce corpus bundle and manifest.json
- [ ] Include creator-supplied SHA256SUMS (claimed inventory)
- [ ] Publish artifact at origin and record transfer metadata

Verifier/Receiver:
- [ ] Retrieve artifact and store to immutable location
- [ ] Canonicalize files using agreed method
- [ ] Recompute manifest_hash_computed and all relevant file hashes
- [ ] Create handoff-record and record match/mismatch
- [ ] If MATCH, allow execution; if MISMATCH, record HANDOFF FAILURE and stop

Executor:
- [ ] Create execution-record (execution_id, command, environment)
- [ ] Execute only after MATCH and record all outputs
- [ ] Produce evidence bundle and compute evidence file hashes
- [ ] Upload evidence bundle to immutable evidence store and record storage location

Evaluator:
- [ ] Confirm manifest_hash_computed matches the manifest used in evaluation
- [ ] Ingest only evidence bundle(s) stored in the immutable evidence store
- [ ] Validate evidence records against schema and signatures (where configured)
- [ ] Produce comparator_output in separate namespace and include audit trail

Example minimal workflow (commands)
- Creator:
  tar -czf ieh-corpus-v1.0.tar.gz manifest.json cases/ fixtures/ SHA256SUMS
  # Optionally sign: gpg --armor --detach-sign ieh-corpus-v1.0.tar.gz
- Receiver/Verifier:
  mkdir /tmp/ieh && tar -xzf ieh-corpus-v1.0.tar.gz -C /tmp/ieh
  cd /tmp/ieh
  # canonicalize and recompute manifest hash (jq path shown; replace with JCS if available)
  jq -S . manifest.json | sha256sum
  # recompute case file canonical hashes similarly and compare to creator SHA256SUMS
  # write handoff-record-<uuid>.json with the fields above
- Executor (after MATCH):
  ./run-tests --corpus /tmp/ieh --output-dir /tmp/evidence-<execution-id>
  tar -czf evidence-bundle-<execution-id>.tar.gz /tmp/evidence-<execution-id>
  sha256sum evidence-bundle-<execution-id>.tar.gz > evidence-bundle-<execution-id>.sha256
  upload to evidence store and record URL in execution-record

Failure modes and remediation (short)
- MISMATCH on manifest hash:
  - Do NOT execute. Notify Creator. Provide handoff-record and raw computed manifest hash. Creator may reissue bundle, or provide an independent mirror, or provide signed proof of origin. All steps logged.
- Missing provenance items in evidence:
  - Evaluator returns UNDETERMINED. Preserve evidence. Request replay or independent confirmation via replay or additional independent sources.
- Evaluator detects independence failure:
  - Record the failure and label any affected comparator_output as not independently verifiable. Do not accept such outputs for critical decisions until independence restored by policy or re-run.

Versioning and immutability of the handoff protocol
- The handoff protocol is itself versioned. Any change to the protocol requires a new protocol manifest version and re-freeze.
- Hand off a copy of the protocol manifest with any corpus bundle so evaluators and implementers can confirm which protocol version was used.

Closing note
- The handoff protocol enforces that: creator claims ≠ verifier truth. Verification is the required step that gates execution. Evidence provenance must be recorded at every stage and stored immutably. Evaluators consume facts only and must return UNDETERMINED in the face of missing or unverifiable provenance rather than manufacturing evidence.

If you want, I can now:
- Produce a concrete handoff-record JSON example filled with placeholders you can copy-and-edit.
- Produce a small shell script that performs the canonicalization + manifest recompute + handoff-record creation (toolchain: jq + sha256sum), or
- Produce the same README formatted as a file ready to paste into the repo at 01-corpus/README.md.

Which of those three would you like next?