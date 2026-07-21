# CASEF v0.6.1 rater-record contract

## 1. Purpose and authority

One `rater_record` represents one human-assessment execution by one exact rater under one exact immutable rater-protocol version over one exact evidence presentation. Multiple raters require separate rater records.

A rerating by the same rater is a new assessment execution and requires a new record when the protocol version, evidence input, evidence order, redaction or transformation, blinding state, displayed metadata, assessment instructions, or assessment execution changes.

Authority is ordered as follows:

1. [`docs/canonical_evidence_contract.md`](../docs/canonical_evidence_contract.md) governs record ownership and evidence-chain boundaries.
2. [`docs/gates.md`](../docs/gates.md) governs qualification and gate semantics.
3. The exact test owns its human-assessment requirements.
4. The exact frozen eligible [`rater protocol`](../spec/rater_protocol_contract.md) governs assessment method and permitted outputs.
5. This contract governs the observed human-assessment execution record.

This is a documentation-level contract only. It creates no protocol instance, rating execution, JSON Schema, adjudication, evidence, or qualification capability.

## 2. Record ownership boundary

A `rater_record` owns:

- one observed human-assessment execution;
- exact rater-protocol binding;
- exact rater identity and eligibility or independence attestation;
- exact immutable assessment inputs;
- exact evidence presentation actually shown;
- assessment execution status;
- exact protocol responses;
- protocol-bound human findings;
- optional protocol-bound individual severity; and
- assessment-output provenance.

A `rater_record` must not own test semantics, planned execution facts, run observations, deterministic findings rewritten as human findings, protocol definitions, multi-rater adjudication, final severity, qualification outcome, policy consequence, or gate authority.

## 3. Required field dictionary

| Field | Requirement | Meaning |
|---|---|---|
| `schema_version` | Required, non-null | Documentation or executable schema version used for the record |
| `rater_record_id` | Required, non-null | Stable identity of one human-assessment record lineage |
| `record_version` | Required, non-null | Exact version in the record lineage; a terminal version is immutable |
| `rater_protocol_reference` | Required | Exact immutable eligible protocol identity, version, reference, hash, and commit |
| `assessment_requirement_references` | Required, non-empty collection | Exact human-assessment requirements being executed and their owning contracts |
| `test_reference` | Conditionally required | Exact test identity, version, reference, and hash when test-governed or intended for canonical qualification use |
| `manifest_reference` | Conditionally required | Exact manifest identity, version, reference, and hash for canonical qualification use or manifest-owned requirements |
| `run_record_references` | Required, non-empty collection | Exact immutable run-record versions and hashes assessed |
| `validation_record_references` | Required collection | Exact validation-record versions and hashes presented or otherwise authorized; may be empty |
| `source_artifact_references` | Required collection | Exact prompt, output, trace, or authorized derivative artifacts used |
| `assessment_input_references` | Required, ordered, non-empty collection | Exact immutable record, artifact, test, manifest, and protocol inputs authorized for assessment |
| `assessment_input_hash` | Required, non-null | Digest binding the ordered input set plus protocol and presentation bindings |
| `evidence_presentation_reference` | Required | Exact immutable artifact or package actually shown to the rater |
| `evidence_presentation_provenance` | Required, structured | Ordered content, visibility, transformation, blinding, metadata, annotations, tool, and process facts |
| `rater_identity` | Required, structured | Stable rater identifier, role, and required qualification references |
| `rater_eligibility_attestation` | Required, structured | Whether protocol-defined eligibility was established and supporting references |
| `rater_independence_disclosure` | Required, structured | Protocol-relevant role overlap, conflict, and prior-exposure facts |
| `assessment_started_timestamp_utc` | Required | Actual canonical UTC assessment-start timestamp |
| `assessment_ended_timestamp_utc` | Required | Actual canonical UTC assessment-end timestamp |
| `assessment_execution_status` | Required, controlled | One of `COMPLETED`, `INCOMPLETE`, `CANCELLED_AFTER_START`, or `PROCESS_ERROR` |
| `protocol_responses` | Required, ordered collection | Exact protocol-item responses; may be empty only when process status and protocol permit |
| `human_findings` | Required collection | Protocol-admitted human findings; may be empty |
| `individual_severity_assessment` | Conditionally permitted | One protocol-bound rater severity assessment; otherwise null or absent |
| `assessment_output_artifact_references` | Required collection | Exact assessment logs or outputs; may be empty when none are produced |
| `recorded_by` | Required, structured | Stable actor or tool identifier and role responsible for record capture |
| `record_created_timestamp_utc` | Required | Canonical UTC timestamp when the record version was created |
| `supersedes_rater_record_reference` | Optional | Exact prior record version superseded by a metadata correction |
| `supersession_reason` | Conditionally required | Bounded reason when supersession is present; otherwise null or absent |

Canonical serialization, timestamp syntax, hash algorithm, hash representation, and canonicalization are governed by [`docs/canonical_serialization_contract.md`](../docs/canonical_serialization_contract.md). The future executable rater-record schema defines this record's required serialized subfields under that authority. Empty strings and sentinel strings must not substitute for null or field absence.

## 4. Exact protocol and input binding

`rater_protocol_reference` binds:

- `rater_protocol_id`;
- `rater_protocol_version`;
- `protocol_reference`;
- `protocol_hash`; and
- `protocol_commit`.

The protocol must be exact, frozen, and eligible for canonical assessment use.

`test_reference` is required when the referenced run is governed by a CASEF test, the human-assessment requirement is test-owned, or canonical qualification use is intended. It may be absent only for a non-test-governed diagnostic assessment. A test identity must not be invented or inferred.

`manifest_reference` is required for canonical qualification use and whenever the assessment requirement or evidence eligibility depends on manifest-owned facts.

`run_record_references` binds exact immutable record versions and hashes and must not silently mix planned and unplanned diagnostic runs. `validation_record_references` may be empty when validation evidence was neither required nor presented; its presence does not imply deterministic results were shown unless the evidence presentation records that fact.

`source_artifact_references` binds exact prompt, output, trace, or authorized derivative artifacts actually used. `assessment_input_references` preserves the exact ordered set of immutable records, artifacts, test and manifest where applicable, and protocol inputs authorized for assessment.

`assessment_input_hash` binds that ordered input set plus protocol and presentation bindings. Its hash algorithm, hash representation, domain separation, and canonicalization are governed by [`docs/canonical_serialization_contract.md`](../docs/canonical_serialization_contract.md). The future executable rater-record schema defines the exact assessment-input object and required bindings under that authority. A digest proves integrity, not correctness, authenticity, expertise, or authority.

## 5. Canonical and diagnostic use

A rater record over any unplanned diagnostic run may be retained for diagnostics but cannot become canonical qualification evidence or be retroactively promoted by adding a later manifest.

One rater record must not mix canonical planned evidence and unplanned diagnostic evidence.

## 6. Exact evidence presentation

`evidence_presentation_reference` binds the exact immutable artifact or package the rater actually saw. `evidence_presentation_provenance` records:

- ordered source references and presentation order;
- displayed and omitted content;
- redaction and transformation state;
- transformation-policy reference where applicable;
- blinding or randomization state;
- model or target metadata visibility;
- deterministic-result visibility;
- annotations shown; and
- presentation tool or process and version.

A list of underlying source artifacts alone is insufficient to establish the actual evidence presentation. A public derivative cannot silently replace canonical source evidence. The rater must not claim assessment of evidence absent from the exact presentation.

## 7. Rater identity and accountability

`rater_identity` uses a stable rater identifier, role, and protocol-required qualification or credential reference where applicable. Public operational content must not use a personal name.

`rater_eligibility_attestation` records whether protocol-defined eligibility was established and its supporting references. `rater_independence_disclosure` records protocol-relevant role overlap, conflicts, and prior exposure.

The record does not decide whether unresolved eligibility is acceptable for qualification. Downstream eligibility and inclusion remain governed by the applicable gate policy and `qualification_record`.

## 8. Assessment execution status

`assessment_execution_status` uses exactly:

- `COMPLETED`;
- `INCOMPLETE`;
- `CANCELLED_AFTER_START`; and
- `PROCESS_ERROR`.

If assessment never begins, no `rater_record` is created; the missing required assessment remains an unmet downstream requirement. `COMPLETED` means only that the protocol-defined assessment execution reached its terminal state. It does not mean favorable behavior, absence of an adverse finding, qualification, or use permission.

`INCOMPLETE`, `CANCELLED_AFTER_START`, and `PROCESS_ERROR` are assessment-process states, not adverse model findings.

## 9. Protocol responses

`protocol_responses` is a required ordered collection. Each response binds:

- `assessment_item_id`;
- `response_value`;
- `response_status`;
- `evidence_references`; and
- `bounded_rationale`.

The permitted `response_value` and `response_status` vocabulary is owned by the exact rater protocol. There is no global generic PASS/FAIL response field.

Responses cannot cite hidden reasoning, chain of thought, evidence not presented, a different run, inferred model identity, or unauthorized external evidence.

## 10. Human findings

`human_findings` is required and may be empty. Every admitted human finding includes:

- `finding_or_construct_id`;
- `finding_or_construct_version`;
- `protocol_rule_reference`;
- `supporting_response_references`;
- `evidence_references`; and
- `bounded_human_statement`.

Every finding must be admitted by an exact protocol rule. A candidate vocabulary label is not sufficient by itself. Deterministic findings must not be copied or relabeled as human findings, and stimulus annotations are not adverse model findings.

A protocol response does not automatically become a finding. A human finding does not automatically create severity, blocking, qualification, or policy consequence.

## 11. Individual severity

`individual_severity_assessment` is permitted only when the exact protocol defines it and is absent or null otherwise. It uses only `S1`, `S2`, `S3`, `UNDETERMINED`, or `NOT_APPLICABLE` and binds exact supported human findings, evidence, and rubric.

It remains one rater's protocol-bound assessment. It is not final adjudicated severity and does not automatically determine qualification.

## 12. Multi-rater boundary

One record represents one rater only. It must not contain a consensus result, majority vote, disagreement resolution, final finding adjudication, final severity, applied gate rule, qualification outcome, or policy consequence.

Multiple rater records remain source-distinct. Their inclusion, exclusion, comparison, disagreement resolution, and final adjudication belong to `qualification_record`.

## 13. Immutability and reassessment

A terminal rater-record version is immutable. A metadata correction creates a new immutable record version with `supersedes_rater_record_reference` and `supersession_reason`; the prior version remains discoverable.

A new assessment execution, not merely a metadata correction, is required when any assessment-relevant input changes, including protocol, rater, source evidence, evidence presentation, ordering, redaction, blinding, instructions, or response items.

## 14. Explicitly prohibited mechanisms

This contract prohibits:

- `hard_fail` or `hard_fail_type`;
- `NO_DEPLOY`;
- global PASS/FAIL qualification;
- automatic human-finding-to-gate mapping;
- automatic individual-severity-to-outcome mapping;
- multi-rater adjudication;
- final severity;
- `qualification_outcome`;
- `policy_consequence`; and
- hidden-reasoning or chain-of-thought assessment.
