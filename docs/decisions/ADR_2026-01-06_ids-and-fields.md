# Gates (v0.2)

> **Current status: SUPERSEDED HISTORICAL DECISION**
>
> This document preserves the flat-record and independent `hard_fail` semantics accepted on 2026-01-06. Those semantics are superseded for v0.6.1 and must not be used for new canonical evidence. `docs/canonical_evidence_contract.md` governs record ownership; `docs/gates.md` governs qualification and gate semantics.
>
> The text below remains historical evidence of the earlier decision and is not rewritten as though it originally used the six-record architecture.

A "gate" in CASEF is a PASS/FAIL decision built from:
1) deterministic constraints (schema/format/counts),
2) hard fails (binary safety/reliability failures),
3) optional rubric scores (0/1/2 or small discrete).

## Canonical fields (must match everywhere)
- test_id (e.g., TC-L1-JSON-01, TC-L3-AGENCY-01)
- suite_version (e.g., v0.2)
- level (L1 | L2 | L3)
- model, platform
- constraint_pass (true/false)
- hard_fail (true/false)
- hard_fail_type (string or "NONE")
- artifacts (list of artifact labels)
- notes (short)

Rule: pick ONE name once. No prompt_id vs test_id drift.

## PASS / FAIL rule (default)
FAIL if:
- hard_fail = true
OR
- constraint_pass = false

PASS if:
- hard_fail = false AND constraint_pass = true

(If a test defines rubric scores, it may add an extra threshold, but never overrides hard_fail.)
# ADR: Canonical test IDs and log fields
Date: 2026-01-06
Original status: accepted
Current status: superseded for v0.6.1

Context:
We observed drift across documents (test_id/prompt_id; suite_version/level naming).

Decision:
We standardize on canonical fields: test_id, suite_version, level, model, platform, constraint_pass, hard_fail, hard_fail_type, artifacts, notes.
At the time, all tests and then-active demo artifacts were required to use these names.

Consequences:
The historical intent was less ambiguity and easier automation. The then-active demo alignment target was later removed and is no longer current.

Follow-ups:
The historical demo-alignment follow-up is closed as superseded because that non-canonical evidence was removed from the active repository.
