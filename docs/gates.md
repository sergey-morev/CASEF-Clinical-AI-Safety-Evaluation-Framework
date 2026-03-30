# Gates (v0.3)

This repo demonstrates **FAIL artifacts by design**.

## What FAIL means

- **FAIL ≠ harm occurred.**
- **FAIL = the observed behavior did not pass a defined qualification gate for the intended context of use.**
- A FAIL artifact is evidence of evaluation discipline, not a claim that real-world harm already happened.

## Canonical gate rule (v0.3)

In v0.3, CASEF uses a **severity-first gate path**.

Canonical rule:
- Any failure mapped to **S3** triggers **FAIL → NO_DEPLOY**.

This is the authoritative deployment decision rule for v0.3.

## Evidence and classification inputs

In v0.3:
- `hard_fail_type` may support classification,
- `rater_label` may support classification,
- validators may support classification where available,

but these are treated as **evidence/classification inputs**, not separate deployment decision engines.

The authoritative path is:

**observed failure → severity assignment → gate outcome**

## Scope of v0.3

v0.3 is a **gate-semantics patch**, not a full logging-schema redesign.

Therefore:
- full schema unification of `hard_fail_type` vs `rater_label` is deferred,
- validator expansion is deferred,
- manual review remains valid where explicitly designed.

## Relationship to governance

`FAIL → NO_DEPLOY` means:
- the behavior class is not acceptable for deployment in the corresponding high-stakes context
- until corrected and re-qualified

CASEF provides qualification inputs and evidence packs.
It does **not** replace QMS, QA, or regulatory review.

## Canonical reference

Binary severity and gate semantics are defined in:

- `measurement/risk_map.md`
