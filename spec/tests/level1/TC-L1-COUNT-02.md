# TC-L1-COUNT-02 — Word-count and surface-format compliance

schema_version: v0.6.1
test_id: TC-L1-COUNT-02
test_version: v0.3-draft
title: Word-count and surface-format compliance
level: L1
construct_id: WORD_COUNT_AND_SURFACE_FORMAT_COMPLIANCE
construct_version: v0.1-draft
spec_status: DRAFT
execution_eligibility: NOT_ELIGIBLE
created_timestamp_utc: 2026-07-17T12:35:34Z

## Migration note

The previous v0.2 text remains available in Git history. It was pre-canonical and is not an approved predecessor record under the v0.6.1 contract.

## Purpose

purpose: Assess bounded word-count and surface-format compliance without evaluating whether the explanation is scientifically complete.

## Prompt protocol

prompt_protocol: Send this single prompt: “In 90–120 words, explain why a failed test condition does not by itself determine a qualification outcome.”

The response must contain 90–120 words inclusive, no bullet lists, no headings, and no disclaimers about being an AI.

## Expected output contract

expected_output_contract: A prose response of 90–120 words inclusive with no bullet-list syntax, heading syntax, or AI-disclaimer language.

## Acceptance criteria

acceptance_criteria: The observable conditions are the inclusive word-count range and absence of bullets, headings, and AI-disclaimer language.

## Deterministic assessment requirements

deterministic_assessment_requirements:

- `V_WORDCOUNT_RANGE`;
- `V_NO_BULLETS`;
- `V_NO_HEADINGS`; and
- `V_NO_AI_DISCLAIMER`.

Exact validator versions are unresolved. This draft remains `NOT_ELIGIBLE`.

## Human assessment requirements

human_assessment_requirements: []
Human assessment is not required for the current construct.

## Applicability

applicability: Single-response tasks that measure only the stated word-count and surface-format constraints.

## Prohibited interpretations

prohibited_interpretations: The result cannot establish factuality, safety, qualification, or broad model capability.
