# CASEF v0.6.1 canonical execution-manifest contract

## 1. Purpose and authority

This document governs the documentation-level field and invariant contract for canonical `execution_manifest` records in CASEF v0.6.1.

`docs/canonical_evidence_contract.md` remains authoritative for record ownership and evidence-chain boundaries. Exact canonical `test_spec` contracts remain authoritative for test semantics. `docs/gates.md` remains the sole authority for evidence eligibility and qualification semantics. This contract is subordinate to all three sources within their respective authority.

This contract does not define JSON Schema, dispatch logic, retry implementation, storage, evidence results, or active gate rules.

## 2. Record ownership boundary

A canonical `execution_manifest` owns:

- planned execution identity and version;
- suite identity and suite version;
- exact selected test identities and versions;
- one planned target and interface;
- the planned execution protocol and conditions;
- repetitions and immutable planned execution slots;
- capture requirements;
- planned deterministic and human assessment requirements; and
- execution authorization state.

An `execution_manifest` must not own:

- test constructs, prompts, acceptance criteria, or other test semantics;
- actual model, product, provider, or interface observations;
- actual timestamps after an execution attempt begins;
- captured prompt or model-output content;
- validator execution results or deterministic findings;
- human findings;
- severity;
- qualification outcomes; or
- policy consequences.

Planned facts remain distinct from the observed facts later owned by `run_record`.

## 3. Field dictionary

| Field | Requirement | Meaning and constraints |
|---|---|---|
| `schema_version` | Required | Exact supported version of this documentation-level contract; distinct from manifest, suite, test, Context-of-Use, and downstream record versions |
| `manifest_id` | Required, non-null | Stable identity of one planned execution definition across its explicit versions |
| `manifest_version` | Required, non-null | Exact version of the planned execution definition; an authorized version is immutable |
| `suite_id` | Required | Stable identity of the selected test suite definition |
| `suite_version` | Required | Human-approved suite version owned by the manifest and bound to the exact `selected_test_references` |
| `execution_class` | Required | Exactly `QUALIFICATION_CANDIDATE` or `PLANNED_DIAGNOSTIC` |
| `context_of_use_reference` | Required for `QUALIFICATION_CANDIDATE` | Exact immutable Context-of-Use ID, version, reference, and hash; optional for `PLANNED_DIAGNOSTIC` only when no qualification claim is made |
| `selected_test_references` | Required, non-empty collection | Exact canonical `test_id`, `test_version`, specification reference, and specification hash for every selected test |
| `planned_target` | Required | One requested target identity, preserving null where an identity element is not exposed rather than inferring it |
| `planned_interface` | Required | Exact planned interface or operating surface, distinct from later observed interface facts |
| `planned_execution_conditions` | Required | Controlled and declared conditions necessary to interpret the planned execution |
| `planned_generation_settings` | Required | Exact controlled generation settings, with explicit not-applicable or unknown handling where the interface does not expose control |
| `planned_tool_conditions` | Required | Planned tool availability and permissions relevant to the selected test contracts |
| `planned_session_conditions` | Required | Planned conversation, memory, and session-state conditions relevant to execution |
| `planned_run_slots` | Required, non-empty collection | Immutable planned repetitions, each with one unique `run_slot_id` and one exact selected test reference |
| `capture_requirements` | Required | Required source-artifact and provenance capture for each planned slot; does not contain captured evidence |
| `validation_requirements` | Required collection | Planned deterministic validation required by selected test specs; references rather than redefines validator semantics |
| `rater_requirements` | Required collection | Planned human assessment required by selected test specs and exact protocols; references rather than redefines rater semantics |
| `manifest_status` | Required | Exactly `DRAFT`, `AUTHORIZED`, `SUPERSEDED`, or `CANCELLED` |
| `prepared_by` | Required | Identity and role of the person or tooling that prepared the manifest; preparation is not execution or qualification authorization |
| `authorized_by` | Required for `AUTHORIZED` and any later lifecycle version of an authorized plan | Identity and role of the human authority approving the exact execution plan; null or absent only when the plan was never authorized |
| `prepared_timestamp_utc` | Required | Canonical UTC timestamp at which this manifest version was prepared |
| `authorized_timestamp_utc` | Required for `AUTHORIZED` and any later lifecycle version of an authorized plan | Canonical UTC timestamp of execution-plan authorization; null or absent only when the plan was never authorized |
| `supersedes_manifest_version` | Optional | Earlier version of the same `manifest_id` replaced by this version; null or absent when none exists |
| `supersession_reason` | Conditionally required | Bounded reason for supersession; required when `supersedes_manifest_version` is present and otherwise null or absent |

The future executable schema must define serialization, timestamp syntax, and controlled subfields. String sentinels such as `"NONE"`, `"N/A"`, or `"UNKNOWN"` must not substitute for null or field absence.

## 4. Selected-test binding

Every selection binds one exact canonical test version through `test_id`, `test_version`, a content reference, and a content hash. A selected test must be `FROZEN` and `ELIGIBLE` under `spec/test_spec_contract.md` when the manifest is authorized.

The manifest does not copy or redefine the construct, prompt protocol, output contract, acceptance criteria, or assessment semantics. Fixed A/B, ordered multi-turn, and closed variant-set protocols remain one selected test version and one planned repetition; their internal steps do not become separate test identities.

`suite_version` and the exact selection set remain bound in the same manifest. A suite label or filename does not establish composition without those exact references.

## 5. Execution classes

The manifest `execution_class` vocabulary is exactly:

| Value | Meaning | Qualification use |
|---|---|---|
| `QUALIFICATION_CANDIDATE` | Planned execution that may later contribute evidence to one bounded qualification decision if every downstream eligibility requirement is satisfied | Potentially eligible; never qualified merely by manifest authorization |
| `PLANNED_DIAGNOSTIC` | Planned execution for debugging, research, or protocol development | Not canonical qualification evidence |

`UNPLANNED_DIAGNOSTIC` is an execution classification, not a third manifest class, because no canonical manifest owns an execution outside an authorized immutable manifest. Such an execution may be retained for diagnostics but cannot contribute canonical qualification evidence.

A `PLANNED_DIAGNOSTIC` execution cannot be retroactively promoted into qualification evidence by later assigning a different class, manifest, or Context of Use.

## 6. Manifest status and authorization

The `manifest_status` vocabulary is exactly:

| Value | Meaning | May start new planned execution? |
|---|---|---|
| `DRAFT` | Mutable execution plan not yet authorized | No |
| `AUTHORIZED` | Exact immutable execution-plan version approved for its declared execution class | Yes, subject to all declared preconditions |
| `SUPERSEDED` | Preserved version replaced through an explicit additive supersession relation | No |
| `CANCELLED` | Preserved plan withdrawn from new execution | No |

Only an exact immutable `AUTHORIZED` manifest version may support planned `QUALIFICATION_CANDIDATE` execution. Authorization freezes all owned planning facts, including target, interface, conditions, selections, requirements, and slots. A change to any of them requires an explicit new manifest version or manifest identity, as applicable.

Supersession or cancellation must preserve the previously authorized version and any attempts already bound to it. A lifecycle change after authorization is recorded additively; it must not silently rewrite an immutable version.

Manifest authorization approves an execution plan only. It is not authorization of a finding, severity, qualification outcome, or policy consequence.

## 7. Planned execution slots

Every planned repetition has one immutable `run_slot_id` that is unique across canonical manifests. Each slot binds:

- the exact manifest identity and version;
- one exact selected `test_id` and `test_version`; and
- the planned repetition represented by that slot.

A future `run_record` references one exact `run_slot_id` together with its manifest binding. One slot cannot silently bind multiple execution attempts. If a new attempt is needed after execution begins, it requires a new run slot or a new manifest version according to the future executable retry policy.

If a precondition fails before an execution attempt begins, no `run_record` is created. The unfilled required slot remains visible as incomplete planned evidence downstream and must not be converted into adverse model behavior.

This contract does not define retry control flow, allocation storage, or runner behavior.

## 8. Context-of-Use binding

Every `QUALIFICATION_CANDIDATE` manifest requires one exact immutable `context_of_use_reference` governed by `spec/context_of_use.md`. The planned target, interface, conditions, and selected tests must be interpretable within that bounded Context of Use.

A `PLANNED_DIAGNOSTIC` manifest may omit the Context-of-Use reference only when no qualification claim is made. Assigning a Context of Use later does not retroactively make diagnostic execution canonical qualification evidence.

## 9. Planned requirements and downstream facts

`capture_requirements`, `validation_requirements`, and `rater_requirements` state what must later be produced or performed. They reference the exact selected test contracts and applicable protocols without copying their semantics.

An absent required run, capture, validation, or rating remains an unresolved downstream evidence requirement. It does not become a model finding, test-constraint failure, or qualification outcome inside the manifest.

The manifest must never contain observed execution timestamps, captured artifact content, conformance results, deterministic or human findings, severity, qualification outcome, or policy consequence.

## 10. Current v0.6.1 implementation status

This document defines a contract only. It creates no JSON Schema, manifest instance, runner, retry policy, validator, rater protocol, evidence, or qualification capability.
