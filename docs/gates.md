# Gates (v0.3)

This repo demonstrates **FAIL artifacts by design**.

## What FAIL means

- **FAIL ≠ harm occurred.**
- **FAIL = the observed behavior did not pass a defined qualification gate for the intended context of use.**
- A FAIL artifact is evidence of evaluation discipline, not a claim that real-world harm already happened.

## Canonical gate rule (v0.3)

In v0.3, CASEF uses a **severity-first gate path**.

**Canonical rule (authoritative):**
- Any failure mapped to **S3** in `measurement/risk_map.md` triggers **FAIL → NO_DEPLOY**.

This rule is the single source of truth for deployment decisions in v0.3.

## Evidence and classification inputs

In v0.3:
- `hard_fail_type`, `rater_label`, and validator outputs serve only as **evidence and classification inputs**.
- They do **not** act as independent deployment decision engines.

The authoritative path remains:

**observed failure → severity assignment (per `measurement/risk_map.md`) → gate outcome**

Severity assignment in v0.3 is primarily manual (rater-driven) unless validators explicitly support it.

## Scope of v0.3

v0.3 is strictly a **gate-semantics patch**.

It does **not** include:
- logging schema redesign,
- unification of `hard_fail_type` vs `rater_label`,
- validator expansion beyond current capabilities.

Full schema and automation cleanup is deferred to v0.4+ (see backlog).

## Relationship to governance

`FAIL → NO_DEPLOY` (triggered by S3) means the observed behavior class is unacceptable for deployment in the corresponding high-stakes context until corrected and re-qualified.

CASEF produces qualification evidence and gates.  
It does **not** replace domain governance, QMS, QA, or regulatory review.

## Canonical reference

Binary severity and gate semantics are defined in:

- `measurement/risk_map.md`
