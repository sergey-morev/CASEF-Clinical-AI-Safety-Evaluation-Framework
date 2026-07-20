# CASEF v0.6.1 run-record contract

## 1. Purpose, authority, and status

One `run_record` represents one observed execution attempt after that attempt has begun. It preserves observed facts and captured source-artifact bindings for that attempt; it does not decide whether the attempt conformed to a plan or whether the model behavior was acceptable.

Authority order is:

1. `docs/canonical_evidence_contract.md` governs ownership and evidence-chain boundaries.
2. `spec/execution_manifest_contract.md` governs planned execution.
3. The exact `test_spec` governs test semantics.
4. `docs/gates.md` governs evidence eligibility and qualification semantics.
5. This document is subordinate within those authority boundaries.

This is a documentation-level field and invariant contract. It is not JSON Schema, storage, a runner, an example record, executable capture, or qualification capability.

## 2. Record ownership boundary

A `run_record` owns:

- one observed execution attempt;
- actual start and end timestamps;
- actual execution and capture status;
- observed target and interface facts;
- observed execution, generation, tool, and session conditions;
- exact source-artifact references and hashes;
- capture provenance; and
- the observed relationship to a planned slot where applicable.

A `run_record` must not own:

- test semantics;
- manifest planning facts;
- manifest-conformance verdicts;
- deterministic findings;
- human findings;
- severity;
- qualification outcomes;
- policy consequences; or
- provider-wide or model-wide conclusions.

Planned facts remain owned by `execution_manifest`. Observed facts remain distinct even when their values later compare equal to the plan.

## 3. Field dictionary

| Field | Requirement | Meaning and constraints |
|---|---|---|
| `schema_version` | Required | Exact supported version of this run-record contract; distinct from all test, manifest, record, protocol, Context-of-Use, validator, and gate-policy versions |
| `run_record_id` | Required, non-null | Unique identity of one observed execution-attempt record lineage; filenames and timestamps are not authoritative identity |
| `record_version` | Required, non-null | Exact version within the run-record lineage; a terminal version is immutable |
| `execution_origin` | Required | Exactly `PLANNED` or `UNPLANNED_DIAGNOSTIC` |
| `manifest_reference` | Required for `PLANNED` | Exact manifest identity, version, reference, and hash; absent for `UNPLANNED_DIAGNOSTIC` |
| `run_slot_id` | Required for `PLANNED` | Exact immutable manifest-owned slot for this attempt; absent for `UNPLANNED_DIAGNOSTIC` |
| `test_reference` | Required for an attempt governed by a CASEF test | Exact test identity, version, reference, and hash; it binds a planned attempt to its selected test and may be absent only for an unplanned diagnostic not governed by a CASEF test |
| `observed_target` | Required structure | Literal target facts exposed by the interface or API, with null or field absence where a fact was hidden or unexposed |
| `observed_interface` | Required structure | Actual interface or operating surface observed during the attempt |
| `observed_execution_conditions` | Required structure | Observed execution conditions relevant to interpretation or reproducibility |
| `observed_generation_settings` | Required structure | Observed generation settings where exposed; it must not copy planned settings as though observed |
| `observed_tool_conditions` | Required structure | Observed tool availability and permissions where exposed or captured |
| `observed_session_conditions` | Required structure | Observed conversation, memory, and session-state facts where exposed or captured |
| `execution_started_timestamp_utc` | Required | Canonical UTC timestamp at which the execution attempt began |
| `execution_ended_timestamp_utc` | Required | Canonical UTC timestamp at which the attempt ended or was observed to have ended |
| `execution_status` | Required | Exactly `COMPLETED`, `PARTIAL_RESPONSE_OBSERVED`, `NO_RESPONSE_OBSERVED`, `EXECUTION_ERROR`, or `CANCELLED_AFTER_START` |
| `capture_status` | Required | Exactly `COMPLETE`, `PARTIAL`, or `FAILED` |
| `prompt_artifact_references` | Required, ordered, non-empty collection | Exact sent prompt or message artifacts, each with artifact reference and hash; raw prompt content is not copied into this record |
| `output_artifact_references` | Required collection | Exact captured response artifacts, each with artifact reference and hash; non-empty when response material was observed and captured |
| `trace_artifact_references` | Required collection | Additional actually captured tool events, interaction traces, or interface metadata; may be empty when no additional trace was required or exposed |
| `capture_provenance` | Required | Capture method, capturing tool or process where applicable, source-artifact creation relationship, available platform timestamps, and integrity-relevant provenance |
| `recorded_by` | Required | Stable actor or tool identifier and role identifier; no personal name in operational public content |
| `record_created_timestamp_utc` | Required | Canonical UTC timestamp at which this record version was created |
| `supersedes_run_record_reference` | Optional | Exact immutable reference to the prior run-record version corrected by this version; null or absent when none exists |
| `supersession_reason` | Conditionally required | Bounded reason for supersession; required when `supersedes_run_record_reference` is present and otherwise null or absent |

The future executable schema defines serialization, timestamp syntax, and required subfields. Empty strings and sentinel strings such as `"NONE"`, `"N/A"`, or `"UNKNOWN"` must not substitute for null or field absence.

## 4. Identity, origin, and planned-slot binding

`run_record_id` identifies the observed execution-attempt record lineage. `record_version` is separate from schema, test, manifest, Context-of-Use, validator, and gate-policy versions.

The `execution_origin` vocabulary is exactly:

| Value | Meaning | Manifest and qualification use |
|---|---|---|
| `PLANNED` | An attempt under one exact immutable execution manifest and one exact planned slot | Requires `manifest_reference` and `run_slot_id`; its later evidence eligibility remains downstream |
| `UNPLANNED_DIAGNOSTIC` | An attempt outside an authorized immutable manifest | May be retained for diagnostics but cannot become canonical qualification evidence |

A `PLANNED` run binds the exact manifest identity, version, reference, and hash; one exact `run_slot_id`; and the exact test identity, version, reference, and hash used for the attempt. One planned slot cannot silently represent multiple attempts.

`UNPLANNED_DIAGNOSTIC` must not use an invented manifest or planned slot. A later manifest cannot retroactively promote an unplanned execution into canonical qualification evidence.

If a precondition fails before the attempt begins, no `run_record` is created. The unfilled slot remains missing planned evidence and is not adverse model behavior.

## 5. Execution and capture status

The `execution_status` vocabulary is exactly:

| Value | Observed meaning |
|---|---|
| `COMPLETED` | The attempt reached an observed terminal interaction state with a complete response interaction as captured, subject to independent capture status |
| `PARTIAL_RESPONSE_OBSERVED` | Some response material was observed, but the response interaction did not reach an observed complete state |
| `NO_RESPONSE_OBSERVED` | No response material was observed for the started attempt |
| `EXECUTION_ERROR` | An interface, transport, tooling, or other execution error was observed after the attempt began |
| `CANCELLED_AFTER_START` | The attempt was cancelled after it began |

The `capture_status` vocabulary is exactly:

| Value | Observed meaning |
|---|---|
| `COMPLETE` | Required source artifacts for the observed attempt were captured completely |
| `PARTIAL` | Some required source-artifact material was not captured completely |
| `FAILED` | Required capture did not complete |

These values describe an attempt and its capture only. `COMPLETED` does not mean a test condition was satisfied, a validator completed, a qualification was issued, behavior was safe, or use is permitted. Partial or failed capture is an evidence defect, not an adverse model finding.

## 6. Observed target and conditions

`observed_target` preserves only what the interface or API actually exposed. Where exposed, it distinguishes reported model name, moving alias, product, provider, and interface. A hidden or unexposed identity remains null or absent where permitted; the record must not infer a hidden model identity or let an assigned `model_family` replace the observed reported identity.

Planned target identity and observed target identity are different facts. A `run_record` stores observed values only and does not decide whether they conform to a manifest.

`observed_interface`, `observed_execution_conditions`, `observed_generation_settings`, `observed_tool_conditions`, and `observed_session_conditions` contain structured observed facts. They must not silently copy planned conditions as observations. A discrepancy may later be assessed by deterministic validation, but `run_record` does not decide manifest conformance.

## 7. Artifact references and capture provenance

After an execution attempt begins, `prompt_artifact_references` is ordered and non-empty. It binds the exact messages or prompt artifacts actually sent, using artifact reference and hash. It does not duplicate complete raw prompt content.

`output_artifact_references` is a required collection and is non-empty whenever response material was observed and captured. It may be empty only when `execution_status` or `capture_status` explains why no output artifact exists. It does not duplicate complete raw output content.

`trace_artifact_references` is a required collection that may be empty when no additional trace was required or exposed. It can reference tool events, interaction traces, or interface metadata only when actually captured.

A public redacted derivative cannot replace a canonical source artifact. Any derivative must remain separately identified and linked to its source under the applicable artifact policy.

`capture_provenance` identifies the capture method, the capturing tool or process where applicable, the source-artifact creation relationship, available platform timestamps, and integrity-relevant provenance. `recorded_by` uses a stable actor or tool identifier and a role identifier such as `EXECUTION_OPERATOR`; it does not use a personal name in operational public content.

## 8. Immutability, correction, and downstream effects

A terminal run-record version is immutable. A correction to observed target, timestamps, execution status, observed conditions, artifact bindings, or capture provenance creates a new immutable `record_version` with `supersedes_run_record_reference` and `supersession_reason`. The prior version remains discoverable.

A changed run-record input requires new downstream validation and any required reassessment. Correction is additive; it never silently rewrites the factual history of the earlier version.

## 9. Prohibited mechanisms and interpretations

The run-record contract must not contain:

- `constraint_pass`;
- `hard_fail` or `hard_fail_type`;
- generic `PASS` or `FAIL` qualification shortcuts;
- `NO_DEPLOY`;
- severity;
- `qualification_outcome`;
- `policy_consequence`;
- manifest-conformance verdicts;
- human ratings; or
- deterministic findings.

Infrastructure, interface, and capture failures are not model findings. A run record cannot support provider-wide, model-wide, product-wide, permanent-model, or general-safety conclusions.

## 10. Current v0.6.1 implementation status

This contract defines ownership and invariants only. It creates no JSON Schema, storage, runner, executable capture, source artifact, model execution, evidence, or qualification capability.
