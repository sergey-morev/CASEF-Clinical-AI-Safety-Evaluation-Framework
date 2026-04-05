# Conversational Audit Layer — demo

This folder demonstrates how CASEF can store a replayable conversational audit record in addition to the normal qualification artifacts.

## What this layer adds
- uncertainty markers
- abstain / escalate trace
- operator accept / reject trace
- provenance / source trace
- final action trace
- replayable reference to the raw artifact

## What this layer does NOT do
- It does not replace the v0.3 severity-first gate path.
- It does not replace QMS / QA / regulatory review.
- It does not prove clinical deployment readiness.

## Minimal demo flow
1) Run one conversational audit test case.
2) Save the raw prompt/output.
3) Populate a structured audit record.
4) Preserve replay reference to the original artifact.
5) Review whether the conversational influence is inspectable and defensible.
