# TC-L1-JSON-01 — Strict JSON conformance

schema_version: v0.6.1
test_id: TC-L1-JSON-01
test_version: v0.3-draft
title: Strict JSON conformance
level: L1
construct_id: STRICT_JSON_CONFORMANCE
construct_version: v0.1-draft
spec_status: DRAFT
execution_eligibility: NOT_ELIGIBLE
created_timestamp_utc: 2026-07-17T12:35:34Z

## Migration note

The previous v0.2 text remains available in Git history. It was pre-canonical and is not an approved predecessor record under the v0.6.1 contract.

## Purpose

purpose: Assess whether an output satisfies the bounded structural requirements for one strict JSON response.

## Prompt protocol

prompt_protocol: Send one prompt requiring a single JSON object with exactly these keys:

- `answer`: a string of no more than 40 words;
- `confidence`: a numeric value from 0 to 1.

The response must contain valid JSON, no extra keys, no Markdown, and no surrounding text.

## Expected output contract

expected_output_contract: One JSON object containing exactly `answer` and `confidence`; `answer` is a string of no more than 40 words, `confidence` is numeric from 0 to 1, and no text or formatting appears outside the object.

## Acceptance criteria

acceptance_criteria: The observable structural conditions are JSON parseability, exact key set, answer word-count limit, numeric confidence range, and absence of surrounding text.

## Deterministic assessment requirements

deterministic_assessment_requirements:

- `V_JSON_PARSE`;
- `V_JSON_KEYS_EXACT`;
- `V_WORDCOUNT`;
- `V_CONFIDENCE_RANGE`; and
- `V_NO_SURROUNDING_TEXT`.

Exact validator versions are not yet bound or implemented. This draft is therefore `NOT_ELIGIBLE`.

## Human assessment requirements

human_assessment_requirements: []
Human assessment is not required for the current construct.

## Applicability

applicability: Single-response tasks whose contract requires the specified strict JSON structure and does not rely on semantic factual review.

## Prohibited interpretations

prohibited_interpretations: Satisfying this test does not establish factual correctness, safety, clinical validity, general reliability, or qualification.
