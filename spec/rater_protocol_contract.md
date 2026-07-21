# CASEF v0.6.1 rater-protocol contract

## 1. Purpose and authority

A rater protocol is a versioned referenced input that defines how one or more human-assessment executions must be conducted. It is not a canonical evidence record, a `rater_record`, a human finding, an adjudication result, a gate policy, or a qualification decision.

Authority is ordered as follows:

1. [`docs/canonical_evidence_contract.md`](../docs/canonical_evidence_contract.md) governs record ownership and evidence-chain boundaries.
2. [`docs/gates.md`](../docs/gates.md) governs qualification and gate semantics.
3. The exact `test_spec` owns the test's human-assessment requirement and required protocol reference.
4. This contract governs rater-protocol identity, content, lifecycle, and execution requirements.
5. [`measurement/rater_record_schema.md`](../measurement/rater_record_schema.md) governs the observed human-assessment execution record.

This is a documentation-level contract only. It does not create a protocol instance, executable workflow, rating surface, evidence, or qualification capability.

## 2. Protocol ownership boundary

A rater protocol owns:

- stable protocol identity and version;
- bounded assessment purpose and construct;
- applicability;
- rater eligibility and independence requirements;
- exact evidence-presentation requirements;
- blinding and randomization requirements where applicable;
- exact assessment instructions and assessment-item identities;
- response scales and controlled vocabularies;
- human-finding admission rules;
- an optional individual-severity rubric;
- required-rater count and independence requirements;
- incomplete-assessment and process-failure handling;
- disagreement and escalation requirements for downstream adjudication;
- quality-control requirements; and
- prohibited interpretations.

A rater protocol must not own observed rating execution, actual evidence presentation, a rater's responses, human findings from an execution, actual disagreement resolution, multi-rater adjudication, final severity, qualification outcome, or policy consequence.

## 3. Required field dictionary

| Field | Requirement | Meaning |
|---|---|---|
| `schema_version` | Required, non-null | Documentation or executable schema version used for the protocol |
| `rater_protocol_id` | Required, non-null | Stable identity of one assessment construct and method |
| `rater_protocol_version` | Required, non-null | Exact protocol version; immutable after approval |
| `title` | Required, non-empty | Human-readable protocol title; not authoritative identity |
| `purpose` | Required, bounded | Assessment purpose and intended use |
| `construct_references` | Required, non-empty collection | Exact constructs assessed and their versions |
| `applicability` | Required, structured | Contexts, tests, evidence, and exclusions to which the protocol applies |
| `input_evidence_requirements` | Required, structured | Eligible immutable records and artifacts required before assessment |
| `rater_eligibility_requirements` | Required, structured | Qualifications, training, calibration, and unresolved-eligibility handling |
| `rater_independence_requirements` | Required, structured | Independence, role-overlap, prior-exposure, and conflict rules |
| `rater_instructions` | Required, versioned | Exact instructions provided to the rater |
| `evidence_presentation_requirements` | Required, structured | Exact content, order, visibility, transformation, and presentation requirements |
| `blinding_and_randomization_requirements` | Required collection | Required blinding or randomization rules; may be empty when not applicable |
| `assessment_items` | Required, ordered, non-empty collection | Exact identified questions or tasks the rater must complete |
| `response_scales` | Required, structured | Protocol-owned response vocabularies and their meanings |
| `human_finding_definitions` | Required collection | Versioned bounded human-finding definitions; may be empty |
| `human_finding_admission_rules` | Required, structured | Exact response and evidence rules for admitting each human finding |
| `individual_severity_requirements` | Required collection | Protocol-owned individual-severity rubric requirements; may be empty |
| `required_rater_count` | Required, positive integer | Number of distinct qualifying assessment executions required downstream |
| `incomplete_assessment_handling` | Required, structured | Treatment of unresolved or incomplete protocol responses |
| `process_failure_handling` | Required, structured | Treatment of assessment interruption, tooling failure, and other process defects |
| `disagreement_and_escalation_requirements` | Required, structured | Conditions for downstream comparison, escalation, or adjudication |
| `quality_control_requirements` | Required, structured | Protocol-defined quality checks and required supporting references |
| `prohibited_interpretations` | Required, non-empty collection | Claims the protocol and its outputs cannot support |
| `protocol_status` | Required, controlled | One of `DRAFT`, `FROZEN`, or `RETIRED` |
| `assessment_eligibility` | Required, controlled | One of `ELIGIBLE` or `NOT_ELIGIBLE` |
| `created_timestamp_utc` | Required | Canonical UTC timestamp for protocol-version creation |
| `approved_timestamp_utc` | Required for `FROZEN` or `RETIRED` | Canonical UTC approval timestamp for the exact approved protocol version; null or absent only for `DRAFT` |
| `approved_by` | Required for `FROZEN` or `RETIRED` | Stable approving-authority identifier and role for the exact approved protocol version; null or absent only for `DRAFT` |
| `supersedes_rater_protocol_version` | Optional | Exact prior version superseded by this version |
| `supersession_reason` | Conditionally required | Bounded reason when a prior version is superseded; otherwise null or absent |

The future executable schema defines serialization, timestamp syntax, and required subfields. Empty strings and sentinel strings must not substitute for null or field absence.

## 4. Identity and versioning

`rater_protocol_id` is the stable identity of one assessment construct and method. Filenames and headings are not authoritative identity.

`rater_protocol_version` identifies the exact protocol version and is distinct from schema, test, construct, record, validator, gate-policy, and Context-of-Use versions. Once approved, that version is immutable.

A material change to any of the following creates a new protocol version:

- rater instructions;
- eligibility or independence requirements;
- evidence presentation or blinding;
- assessment items;
- response vocabulary;
- finding-admission rules;
- individual-severity rubric;
- required-rater count; or
- disagreement handling.

A material change to the assessed construct or assessment method may require a new `rater_protocol_id`, not merely a new version.

## 5. Lifecycle and assessment eligibility

`protocol_status` uses exactly:

- `DRAFT`: mutable, unapproved, and unable to support canonical human assessment; `approved_timestamp_utc` and `approved_by` are null or absent;
- `FROZEN`: exact approved immutable protocol version; `approved_timestamp_utc` and `approved_by` are required and resolvable; and
- `RETIRED`: previously approved immutable protocol version retained for history but unavailable for new assessments; its approval provenance remains required and discoverable.

`assessment_eligibility` uses exactly:

- `ELIGIBLE`; and
- `NOT_ELIGIBLE`.

Only a `FROZEN` protocol may be `ELIGIBLE`. `DRAFT` and `RETIRED` protocols must be `NOT_ELIGIBLE`. A `FROZEN` protocol is not validly eligible unless `approved_timestamp_utc` and `approved_by` are present and resolvable. `FROZEN` and approval do not automatically mean `ELIGIBLE`. Eligibility permits exact protocol selection but does not create evidence eligibility, a human finding, severity, qualification, or policy consequence.

## 6. Rater identity and independence requirements

The protocol must define required domain or task qualifications, training or calibration where applicable, independence requirements, conflict-of-interest disclosure, and treatment of unresolved eligibility or independence. It must state whether a rater may have participated in test design, execution, validation, or prior adjudication.

Public operational content does not require personal names. Canonical records use stable rater IDs plus role or qualification references; any private identity mapping is controlled outside the public repository.

## 7. Evidence-presentation requirements

The protocol must define exactly:

- eligible upstream record and artifact types;
- presentation order;
- prompt, output, and trace visibility;
- whether deterministic findings or validator results are shown;
- redaction and transformation rules, including source-to-derivative mapping;
- blinding and randomization state;
- target or model metadata visibility;
- annotations or contextual metadata shown;
- presentation tool or format requirements; and
- prohibited hidden information.

The actual observed evidence presentation belongs to `rater_record`, not to the protocol. The protocol must prohibit reliance on hidden reasoning, chain of thought, unpresented evidence, inferred model identity, memory of a different run, or external information not authorized by the protocol.

## 8. Assessment items and responses

Each assessment item must define:

- `assessment_item_id`;
- `construct_reference`;
- `instruction`;
- `permitted_response_vocabulary`;
- `required_evidence_citation`;
- `missing_or_indeterminate_handling`; and
- `finding_admission_relationship`.

Response scales are protocol-defined. CASEF does not define a universal global human-rating PASS/FAIL field.

## 9. Human-finding definitions

Every protocol-defined human finding binds a stable finding or construct identifier, version, exact assessment item or response rule, required evidence relationship, bounded meaning, and explicit prohibited interpretations.

A protocol response does not automatically become a human finding. A human finding does not automatically determine severity, qualification, blocking, or policy consequence.

Candidate vocabulary in [`measurement/rater_labels.md`](../measurement/rater_labels.md) remains non-authoritative unless an exact frozen eligible protocol adopts and defines the relevant label.

## 10. Individual severity

`individual_severity_requirements` may be empty when the protocol does not request an individual severity assessment. When individual severity is permitted or required, the protocol must define the exact rubric and may use only `S1`, `S2`, `S3`, `UNDETERMINED`, or `NOT_APPLICABLE`.

Individual rater severity is not final adjudicated severity and must not automatically map to qualification.

## 11. Multiple raters and disagreement

The protocol may define required rater count, independence, ordering or parallelism, disagreement thresholds, and conditions requiring escalation or adjudication. It must not claim that an individual `rater_record` performs multi-rater adjudication.

Actual inclusion, exclusion, disagreement resolution, final severity, and qualification remain downstream in `qualification_record`.

## 12. Explicitly prohibited mechanisms

This contract prohibits:

- `hard_fail` or `hard_fail_type`;
- `NO_DEPLOY`;
- global PASS/FAIL qualification;
- automatic label-to-gate mapping;
- automatic individual-severity-to-outcome mapping;
- hidden-reasoning assessment; and
- unversioned free-form assessment.
