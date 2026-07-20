# CASEF — v0.6.1 one-page overview

**One-liner:** Physician-led framework for structuring evidence for bounded qualification decisions about LLM behavior in clinical-like and other high-stakes settings.

## What CASEF is

CASEF organizes contracts, observations, assessment, and decision semantics for bounded evaluation of LLM behavior. Its core formula is:

**spec → measurement → gates → evidence**

The framework keeps a test contract, observed execution, deterministic validation, human assessment, and authorized qualification decision distinct. Qualification remains scoped to an exact Context of Use, target, interface, date, test contract, and evidence set.

## What CASEF currently provides

- a six-record evidence-chain architecture;
- qualification and gate semantics in [`docs/gates.md`](gates.md);
- documentation-level contracts for [`test_spec`](../spec/test_spec_contract.md), [`execution_manifest`](../spec/execution_manifest_contract.md), [`run_record`](../measurement/run_record_schema.md), [`validation_record`](../measurement/validation_record_schema.md), [`rater_record`](../measurement/rater_record_schema.md), [Context of Use](../spec/context_of_use.md), and [`qualification_record`](../measurement/qualification_record_schema.md);
- a documentation-level [`rater-protocol contract`](../spec/rater_protocol_contract.md); and
- draft current test assets, including [`TC-L1-JSON-01`](../spec/tests/level1/TC-L1-JSON-01.md), [`TC-L1-COUNT-02`](../spec/tests/level1/TC-L1-COUNT-02.md), and [`TC-L3-AGENCY-01`](../framework/level3/agency/TC-L3-AGENCY-01.md).

## What CASEF does not currently provide

- an executable canonical pipeline;
- JSON Schema;
- a runner;
- implemented canonical validators;
- approved rater protocols;
- rating executions;
- an evidence pack;
- current model qualification; or
- clinical deployment readiness.

No current asset establishes a live qualification capability.

All six canonical record contracts are documented. The rater-protocol contract is documented, but no approved rater-protocol instance exists.

## Levels

### Level 1 — Format and constraint robustness

Bounded structural requirements such as JSON shape, word limits, and surface-format constraints.

### Level 2 — Multi-turn stress and semantic invariance

Future work on behavior across repeated turns and defined transformations.

### Level 3 — Value conflicts and agency/stakes

Future protocol-bound work on capability disclosure, responsibility handling, verification boundaries, and safety under pressure.

## Current documentation surfaces

- [`docs/canonical_evidence_contract.md`](canonical_evidence_contract.md) — record ownership and evidence-chain boundaries.
- [`docs/gates.md`](gates.md) — sole qualification and gate-semantics authority.
- [`spec/test_spec_contract.md`](../spec/test_spec_contract.md) — canonical test-specification contract.
- [`spec/execution_manifest_contract.md`](../spec/execution_manifest_contract.md) — canonical planned-execution contract.
- [`measurement/run_record_schema.md`](../measurement/run_record_schema.md) — canonical observed-execution contract.
- [`measurement/validation_record_schema.md`](../measurement/validation_record_schema.md) — canonical deterministic-validation contract.
- [`spec/rater_protocol_contract.md`](../spec/rater_protocol_contract.md) — versioned human-assessment protocol boundary.
- [`measurement/rater_record_schema.md`](../measurement/rater_record_schema.md) — canonical individual human-assessment execution contract.
- [`spec/context_of_use.md`](../spec/context_of_use.md) — bounded Context-of-Use contract.
- [`measurement/qualification_record_schema.md`](../measurement/qualification_record_schema.md) — qualification-record contract.

## MedRLHF Gap backlog concept

Alignment claims may later be recorded as context for qualification. An alignment claim is not qualification evidence. This overview establishes no active MedRLHF-specific test, gate, or current qualification claim.

## Non-goals

- No jailbreak or exploitation instructions.
- No claims about vendor internals or hidden model identity.
- No clinical diagnosis or treatment recommendations.
- No provider-wide, company-wide, product-wide, permanent-model, or general-safety conclusion from bounded evidence.
