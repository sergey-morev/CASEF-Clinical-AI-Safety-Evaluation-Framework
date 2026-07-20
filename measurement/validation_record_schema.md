# CASEF v0.6.1 validation-record contract

## 1. Purpose, authority, and status

One `validation_record` represents one deterministic validator execution over one exact immutable input set. It preserves the validator execution, exact input binding, mechanically established check results, and deterministic findings produced under the exact validator contract.

Authority order is:

1. `docs/canonical_evidence_contract.md` governs ownership and evidence-chain boundaries.
2. Exact test and manifest requirements govern what may be checked.
3. The exact versioned validator contract governs deterministic validator behavior.
4. `docs/gates.md` governs downstream qualification semantics.
5. This document is subordinate within those authority boundaries.

This is documentation-level only. It does not implement a validator, JSON Schema, runner, active gate, evidence, or qualification.

## 2. Record ownership boundary

A `validation_record` owns:

- one deterministic validator execution;
- exact immutable input binding;
- exact validator identity and version;
- validator execution status;
- mechanically established check results;
- deterministic findings produced under the validator contract; and
- validator runtime and output provenance.

A `validation_record` must not own:

- raw model-output content copied from source artifacts;
- human judgment;
- rater labels;
- individual or final human severity;
- final adjudication;
- qualification outcome;
- policy consequence; or
- gate authority.

## 3. Field dictionary

| Field | Requirement | Meaning and constraints |
|---|---|---|
| `schema_version` | Required | Exact supported version of this validation-record contract; distinct from all run, test, manifest, validator, Context-of-Use, and gate-policy versions |
| `validation_record_id` | Required, non-null | Unique identity of one validator-execution record lineage; distinct from run, test, manifest, validator, and qualification identity |
| `record_version` | Required, non-null | Exact version within the validation-record lineage; a terminal version is immutable |
| `run_record_reference` | Required | Exact immutable run-record reference containing `record_type`, `record_id`, `record_version`, `record_reference`, and `record_hash` |
| `test_reference` | Conditionally required | Exact test identity, version, reference, and hash governing the validated requirement; required when the referenced `run_record` is governed by a CASEF test or any executed validation requirement is test-owned; may be absent only for an `UNPLANNED_DIAGNOSTIC` run not governed by a CASEF test |
| `manifest_reference` | Conditionally required | Exact manifest identity, version, reference, and hash when planned execution, manifest-conformance checks, or a manifest-owned requirement is relevant |
| `validator_reference` | Required | Exact `validator_id`, `validator_version`, validator-contract reference and hash, plus executable implementation reference and hash or commit where applicable |
| `validation_requirement_references` | Required, non-empty collection | Exact deterministic requirements being executed and their owning test, manifest, validator, or policy contract |
| `validation_input_references` | Required, ordered, non-empty collection | Exact immutable run, artifact, test, manifest where applicable, and other explicitly authorized input references |
| `validation_input_hash` | Required | Digest binding the exact ordered `validation_input_references` set |
| `validation_started_timestamp_utc` | Required | Canonical UTC timestamp at which the validator invocation began |
| `validation_ended_timestamp_utc` | Required | Canonical UTC timestamp at which the validator invocation ended or was observed to have ended |
| `validator_execution_status` | Required | Exactly `COMPLETED`, `EXECUTION_ERROR`, or `CANCELLED_AFTER_START` |
| `check_results` | Required collection | Structured mechanically established check results under the exact validator contract |
| `deterministic_findings` | Required collection | Mechanically established findings under the exact validator contract; may be empty |
| `validator_output_artifact_references` | Required collection | Exact validator logs or reports, when captured, without copying their complete content into this record |
| `validator_runtime` | Required | Relevant observed validator environment, implementation version, and dependencies required for reproducibility |
| `produced_by` | Required | Stable tooling or role identifier; no personal name in operational public content |
| `record_created_timestamp_utc` | Required | Canonical UTC timestamp at which this record version was created |
| `supersedes_validation_record_reference` | Optional | Exact immutable reference to the prior validation-record version corrected by this version; null or absent when none exists |
| `supersession_reason` | Conditionally required | Bounded reason for supersession; required when `supersedes_validation_record_reference` is present and otherwise null or absent |

The future executable schema defines serialization, timestamp syntax, hash algorithm, canonicalization, and required subfields. Empty strings and sentinel strings such as `"NONE"`, `"N/A"`, or `"UNKNOWN"` must not substitute for null or field absence.

## 4. One validator invocation and exact input binding

One validation record represents one validator invocation against one exact run-record input under one exact bounded requirement set. Multiple separate validator invocations require separate validation records.

`run_record_reference` binds the exact immutable run-record version using `record_type`, `record_id`, `record_version`, `record_reference`, and `record_hash`.
`test_reference` is required when the referenced `run_record` is governed by a CASEF test or when any executed validation requirement is test-owned. It may be absent only for an `UNPLANNED_DIAGNOSTIC` run that is not governed by a CASEF test.
When `test_reference` is absent, every executed requirement must instead bind its exact non-test owning contract through `validation_requirement_references`, together with the exact validator contract. A test identity must not be invented or inferred. A validation record produced under this exception remains diagnostic and cannot contribute canonical qualification evidence.

`manifest_reference` is conditionally required when a planned execution is checked against manifest requirements, manifest-conformance checks are performed, or the applicable validation requirement depends on manifest-owned facts.

`validation_requirement_references` identifies the exact deterministic requirements being executed and the test, manifest, validator, or policy contract that owns each one. `validation_input_references` preserves the exact ordered set of the run-record reference, relevant prompt/output/trace artifacts, test reference, manifest reference where applicable, and other explicitly authorized immutable inputs.

`validation_input_hash` binds that exact ordered input-reference set. The digest proves input-set integrity, not scientific correctness, validator correctness, or authority by itself.

Exact immutable references and hashes, not filenames, establish input and validator identity. Prompt, output, and trace material remain source artifacts referenced by the validation record; their complete raw content is not duplicated there.

## 5. Validator binding and provenance

`validator_reference` includes `validator_id`, `validator_version`, a validator-contract reference and hash, and an executable implementation reference and hash or commit where applicable. This contract does not claim that current validators already exist.

A changed validator contract, implementation, source hash, dependency set, or relevant runtime requires a new validator execution and a new validation record. `validator_runtime` records the relevant observed environment, implementation version, and dependencies needed for reproducibility. `produced_by` uses stable tooling or role identifiers such as `VALIDATOR`; it does not use a personal name in operational public content.

`validator_output_artifact_references` binds exact validator logs or reports when captured. The record references those outputs rather than copying complete output content.

## 6. Validator-execution and check-result status

The `validator_execution_status` vocabulary is exactly:

| Value | Meaning |
|---|---|
| `COMPLETED` | The validator invocation reached an observed terminal state; it does not mean every test condition was satisfied |
| `EXECUTION_ERROR` | The validator invocation encountered an execution, configuration, dependency, or infrastructure error |
| `CANCELLED_AFTER_START` | The validator invocation was cancelled after it began |

If validation never begins, no validation record is created. Missing required validation remains an unmet downstream requirement. `EXECUTION_ERROR` is not an adverse model finding.

Each `check_results` entry identifies:

- `check_id`;
- `validator_rule_id`;
- `requirement_reference`;
- `check_status`;
- `expected_condition`;
- `observed_value_or_reference`;
- `evidence_references`; and
- `bounded_reason`.

The `check_status` vocabulary is exactly:

| Value | Meaning |
|---|---|
| `SATISFIED` | The exact validator mechanically established that the test condition was met |
| `VIOLATED` | The exact validator mechanically established that the test condition was not met |
| `INDETERMINATE` | The exact validator could not establish either state from the authorized input set |
| `NOT_EVALUATED` | The check was not performed and includes an explicit bounded reason |

`SATISFIED` and `VIOLATED` are mechanically established test-condition results. No check status automatically creates severity, qualification, or policy consequence. This contract has no global generic `PASS` or `FAIL` field.

## 7. Deterministic findings and manifest conformance

`deterministic_findings` is a required collection and may be empty. Every admitted deterministic finding includes:

- a stable finding or construct identifier;
- finding or construct version;
- exact supporting check result;
- exact evidence references;
- exact validator reference; and
- a mechanically established bounded statement.

A deterministic finding contains no human inference, severity, or gate consequence. A check violation does not need to become a separate named finding unless the exact validator contract defines that output.

A validation record may establish manifest-conformance checks only when the exact manifest and run records are bound, an exact versioned validator rule defines the mechanical check, and the result remains deterministic. Manifest conformance must never be stored in `run_record`.

## 8. Immutability, correction, and revalidation

A terminal validation-record version is immutable. A correction of validation-record metadata creates a new immutable `record_version` with `supersedes_validation_record_reference` and `supersession_reason`. The prior version remains discoverable.

A new validation execution is required when any relevant input changes, including the run-record reference or version, source-artifact binding, test version, manifest input relevant to the check, validation requirement, validator contract, validator implementation, or runtime condition that affects results.

## 9. Prohibited mechanisms and interpretations

The validation-record contract must not contain:

- human findings;
- rater scores;
- final severity;
- `qualification_outcome`;
- `policy_consequence`;
- `NO_DEPLOY`;
- `hard_fail` or `hard_fail_type`;
- a generic global `PASS` or `FAIL` field; or
- automatic blocking.

A validator failure, missing input, or infrastructure error is not model behavior. A validation record does not authorize qualification or create provider-wide, model-wide, product-wide, permanent-model, or general-safety conclusions.

## 10. Current v0.6.1 implementation status

This contract defines ownership and invariants only. It creates no JSON Schema, validator contract or implementation, runner, executable validation, evidence, active gate, or qualification capability.
