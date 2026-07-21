# CASEF v0.6.1 qualification-record contract

## 1. Purpose, authority, and status

A `qualification_record` represents one immutable, authorized qualification decision execution for one exact Context of Use. It consumes exact immutable upstream records and never rewrites them.

`docs/canonical_evidence_contract.md` governs record ownership and evidence-chain boundaries. `docs/gates.md` remains the sole authority for qualification and gate semantics.

This is a documentation-level field and invariant contract. It is not JSON Schema, executable validation, an active gate policy, or an example record.

## 2. Record boundary

The `qualification_record` owns:

- inclusion and exclusion of evidence;
- adjudication of deterministic and human findings while preserving provenance;
- final severity where applicable;
- one qualification outcome;
- one separate Context-of-Use policy consequence;
- applied gate rules and bounded rationale; and
- decision authority and input provenance.

It references, but does not copy, the Context of Use, manifest, test where applicable, run records, validation records, rater records, source artifacts, and gate policy.

## 3. Field dictionary

| Field | Requirement | Meaning and constraints |
|---|---|---|
| `schema_version` | Required | Exact supported version of this qualification-record contract; distinct from all test, manifest, record, protocol, Context-of-Use, and gate-policy versions |
| `qualification_id` | Required | Unique identity of this immutable authorized decision execution |
| `decision_type` | Required | Bounded decision-scope category whose exact value must resolve in the versioned vocabulary of the applied gate policy bound by `gate_policy_id`, `gate_policy_version`, and `gate_policy_commit`; it must not express provider-wide, company-wide, product-wide, permanent-model, or general-safety scope |
| `context_of_use_reference` | Required, non-null | Exact `context_id`, `context_version`, content reference, and content hash governed by `spec/context_of_use.md` |
| `manifest_reference` | Conditionally required | Exact manifest identity, version, reference, and hash; required for `QUALIFIED` and `NOT_QUALIFIED`; may be null only for `REVIEW_REQUIRED` when the unresolved binding is recorded in `unmet_requirements` |
| `test_reference` | Required where applicable | Exact test ID, test version, reference, and hash when the decision or an applied rule is test-specific |
| `run_record_references` | Required collection | Exact run-record identities, versions where defined, references, and hashes considered by the decision; must be non-empty for a completed positive or negative qualification |
| `validation_record_references` | Required collection | Exact validation-record references and hashes required by the applicable tests and gate policy; unresolved required validation is recorded in `unmet_requirements` |
| `rater_record_references` | Required collection | Exact rater-record references and hashes required by protocol or gate policy; the collection may be empty only when human assessment is not required or remains explicitly unmet under `REVIEW_REQUIRED` |
| `evidence_inclusions` | Required collection | Exact upstream record and artifact references admitted as eligible decision inputs, with inclusion basis |
| `evidence_exclusions` | Required collection | Exact upstream references excluded from the decision, with controlled or bounded exclusion reason; excluded records remain discoverable |
| `unmet_requirements` | Required collection | Policy, evidence, assessment, or authority requirements not established; empty only when all applicable requirements are resolved |
| `adjudicated_findings` | Required collection | Findings considered in the decision, preserving deterministic or human source type, exact source records, evidence references, adjudication status, and bounded rationale |
| `adjudicated_severity` | Required | One of `S1`, `S2`, `S3`, `UNDETERMINED`, or `NOT_APPLICABLE`; must cite supported findings and the applicable rubric when severity is assigned |
| `qualification_outcome` | Required | Exactly one of `REVIEW_REQUIRED`, `QUALIFIED`, or `NOT_QUALIFIED` |
| `policy_consequence` | Required | Exactly one of `USE_PERMITTED`, `USE_PROHIBITED`, or `USE_REQUIRES_REVIEW`; remains distinct from `qualification_outcome` |
| `applied_gate_rules` | Required collection | Exact gate-policy rules considered and their application; `NOT_QUALIFIED` requires at least one explicit applied blocking rule supported by eligible evidence |
| `decision_rationale` | Required | Bounded, evidence-linked explanation of the decision; must not embed raw prompt or output content |
| `record_prepared_by` | Required | Identity and role of the human or tool that prepared the record; preparation is not decision authority |
| `decision_authority` | Required | Identity, PI role, and authorization basis of the person who authorized the qualification decision |
| `decision_started_timestamp_utc` | Required | UTC timestamp at which authorized decision work began |
| `decision_ended_timestamp_utc` | Required | UTC timestamp at which the immutable decision record was completed |
| `gate_policy_id` | Required | Stable identity of the exact applied gate policy |
| `gate_policy_version` | Required | Exact version of the applied gate policy |
| `gate_policy_commit` | Required | Repository commit binding the exact gate-policy text used |
| `decision_input_hash` | Required | Digest of the exact ordered decision-input reference set, Context-of-Use binding, and gate-policy binding; distinct from individual record and artifact hashes |
| `supersedes_qualification_id` | Optional | Identity of a prior qualification record replaced or withdrawn by this decision; null or absent when no predecessor exists |
| `supersession_reason` | Conditionally required | Bounded reason for supersession; required whenever `supersedes_qualification_id` is present and otherwise null or absent |

All required timestamps use one canonical UTC representation defined by the future executable schema. This documentation does not select serialization syntax.

### Decision-type policy binding

CASEF v0.6.1 does not define a separate global `decision_type` enum. The exact value must be recognized by the applied gate-policy version identified by `gate_policy_id`, `gate_policy_version`, and `gate_policy_commit`. An unrecognized or unresolvable value is an unmet policy requirement and normally requires `qualification_outcome: REVIEW_REQUIRED`.

## 4. Exact reference structure

Each upstream record reference contains:

- `record_type`;
- `record_id`;
- `record_version`, where the referenced contract defines one;
- `record_reference`; and
- `record_hash`.

Each artifact reference contains:

- `artifact_reference`; and
- `artifact_hash`.

The record may cite individual fields in an upstream record for rationale or rule application, but it must not copy the complete upstream record or redefine the cited fact. Public redacted derivatives require their own identity and hash plus an explicit source-artifact and transformation-policy relationship.

## 5. Evidence inclusion, exclusion, and unmet requirements

Every upstream record considered by the decision must have one explicit disposition: included or excluded. Inclusion requires established eligibility under the exact gate policy. Exclusion records the exact input and reason without deleting it from history.

An infrastructure failure, missing artifact, failed validator execution, failed human assessment, unresolved required disagreement, or other incomplete requirement normally appears in `unmet_requirements` and produces `REVIEW_REQUIRED`. It must not be represented as adverse model behavior.

Unplanned diagnostic evidence, invalidated upstream records, and public-redacted-only substitutes cannot be included as canonical qualification evidence. Their existence may be recorded as exclusions where relevant.

## 6. Adjudicated findings and severity

Each adjudicated finding preserves whether its source is deterministic or human and cites exact source records and evidence. Adjudication status uses a versioned policy vocabulary; where the policy uses `SUPPORTED`, `NOT_SUPPORTED`, `DISPUTED`, or `UNRESOLVED`, only `SUPPORTED` findings may support final severity or a blocking rule.

Stimulus annotations are not adverse model findings by themselves. Findings with different constructs or versions must not be silently merged.

Final severity uses exactly:

- `S1`;
- `S2`;
- `S3`;
- `UNDETERMINED`; or
- `NOT_APPLICABLE`.

`UNDETERMINED` must not silently become `S1`. `NOT_APPLICABLE` must not represent review that was required but not performed. Severity does not automatically determine qualification unless the exact applied gate policy contains and applies that rule.

## 7. Qualification outcome and policy consequence

Qualification outcome uses exactly:

- `REVIEW_REQUIRED`;
- `QUALIFIED`; or
- `NOT_QUALIFIED`.

Policy consequence uses exactly:

- `USE_PERMITTED`;
- `USE_PROHIBITED`; or
- `USE_REQUIRES_REVIEW`.

Outcome and consequence are separate fields. `QUALIFIED` requires complete eligible evidence and satisfaction of every applicable gate-policy criterion. `NOT_QUALIFIED` requires eligible evidence and at least one explicit applied blocking rule. Missing or failed required evidence normally produces `REVIEW_REQUIRED`.

The consequence is derived under the exact Context-of-Use policy. A research activity may remain permitted while a separately bounded operational use is prohibited.

## 8. Decision authority and provenance

`record_prepared_by` and `decision_authority` must identify different responsibilities even when one person performs both roles. Codex, a runner, validator, or other automated tooling may prepare data or a record, but cannot authorize qualification.

`decision_authority` must identify the PI who authorized the decision and the authority basis. The record also binds the exact gate policy by ID, version, and repository commit.

`decision_input_hash` binds the ordered set of exact inputs used for the decision. Its hash algorithm, hash representation, domain separation, and canonicalization are governed by [`docs/canonical_serialization_contract.md`](../docs/canonical_serialization_contract.md). The future executable qualification-record schema defines the exact decision-input object and required bindings under that authority. The digest proves input-set integrity, not scientific correctness or authority by itself.

## 9. Immutability and supersession

A completed `qualification_record` is immutable. Correction, newly available evidence, changed interpretation, changed gate policy, changed Context of Use, or reevaluation creates a new `qualification_id`.

Supersession identifies the predecessor and reason. The prior record remains immutable and discoverable. A changed model version or moving alias at a later date cannot silently inherit an earlier qualification.

## 10. Structural invariants

- `context_of_use_reference` is always exact, immutable, and non-null.
- `QUALIFIED` has no unresolved policy-required evidence or assessment requirement.
- `NOT_QUALIFIED` cites eligible evidence and an explicit applied blocking rule.
- `REVIEW_REQUIRED` identifies the unresolved requirement or authority condition.
- Every included or excluded input uses exact immutable references and hashes.
- Deterministic and human findings retain their provenance.
- Qualification and policy consequence remain separate.
- The decision authority is an identified PI, not the technical producer or automated tooling.
- Supersession is additive and never overwrites the prior record.

## 11. Prohibited content and mechanisms

The canonical qualification record must not contain:

- `wave_id`;
- a canonical `hard_fail` or `hard_fail_type` mechanism;
- `NO_DEPLOY` as a qualification outcome;
- raw prompt or raw output content;
- complete copied upstream records;
- hidden or inferred provider/model identity;
- provider-wide, company-wide, product-wide, permanent-model, or general-safety claim fields; or
- an automatic severity-to-outcome mapping not present in the exact applied gate policy.

Optional values use actual null or field absence only where this contract permits it. Empty strings and sentinel strings such as `"NONE"`, `"N/A"`, or `"UNKNOWN"` do not represent missing data unless a field explicitly defines one as controlled vocabulary.

## 12. Current v0.6.1 implementation status

This contract does not provide JSON Schema, a validator, a runner, a rater protocol, a gate rule, storage, or a qualification-record instance. CASEF cannot issue a canonical `QUALIFIED` or `NOT_QUALIFIED` record until the required upstream contracts, protocols, tooling, and PI-approved gate rules are implemented and reviewed.
