# CASEF — Clinical AI Safety Evaluation Framework

Physician-led, clinical-grade qualification framework for LLM safety and reliability.

**Core idea:**  
CASEF treats LLM safety as a **qualification problem**, not a vibe, benchmark score, or alignment claim.

It provides a reproducible structure for evaluating model behavior under explicit constraints using:

**spec → measurement → gates → evidence**

CASEF is **model-agnostic** and **platform-portable** across apps, web UIs, and APIs.  
It focuses on **reproducible external evaluation artifacts**, not internal telemetry.

---

## Why CASEF exists

Most LLM evaluation still overweights answer quality and underweights **deployment-relevant failure**.

In real use, especially in clinical and other high-stakes environments, the question is not only:

- *Can the model produce a plausible answer?*

but:

- *Does it stay within role and constraints?*
- *Does it fabricate actions or certainty?*
- *Does it fail safely when information is missing?*
- *Can its behavior be logged, reviewed, and defended?*

CASEF is designed for this layer.

It is not a benchmark leaderboard.  
It is a framework for producing **qualification-grade evidence**.

---

## What CASEF evaluates

CASEF is intended for failures that matter in deployment, not just benchmark performance.

Examples include:

- format violations under explicit constraints
- hallucinated actions or unsupported claims
- unsafe reassurance
- abstain / escalate failures
- role drift under pressure
- inconsistency between stated policy and observed behavior
- artifact-level failures that break reproducibility or reviewability

---

## What makes CASEF different

Unlike benchmark-style evaluation, CASEF focuses on **portable, inspectable qualification artifacts**:

- explicit test specs
- structured logging
- artifact taxonomy
- redaction rules
- model/platform registry
- reproducible evidence packs
- pass/fail-oriented review surfaces

The goal is not to ask whether a model is “good.”  
The goal is to ask whether a behavior is **acceptable for a defined context of use**.

---

## Scope

CASEF is currently designed as a framework for:

- clinical AI reliability evaluation
- high-stakes conversational system qualification
- platform-agnostic behavioral testing
- evidence-oriented safety review
- pre-deployment or controlled evaluation workflows

---

## Non-goals

- No jailbreak content
- No claims about internal model telemetry
- No claims of clinical deployment readiness by default
- Not medical advice
- Not an AI doctor
- Not a productized monitoring platform
- Not a replacement for domain governance, QA, or regulatory review

---

## Current design principles

- **Observed > claimed**
- **External behavior over internal speculation**
- **Reproducibility over intuition**
- **Failure visibility over vague optimism**
- **Qualification over hype**
- **Model-agnostic evaluation over vendor lock-in**

---

## Repository contents (v0.1)

### Core documentation
- `docs/casef_one_pager.md` — one-page overview
- `measurement/log_schema.md` — minimal logging schema
- `measurement/artifact_taxonomy.md` — artifact taxonomy
- `redaction/guide.md` — redaction standard
- `spec/model_registry.md` — model/platform registry
- `prompt_registry.md` — prompt registry

### Execution / validation surface
- `run_suite.py`
- `validators.py`
- `run_manifest.md`
- `README_demo.md`

### Example test assets
- `TC-L1-JSON-01`
- `TC-L1-COUNT-02`
- `TC-L3-AGENCY-01`

### Example result artifacts
- `results.jsonl`
- model/platform-specific captured outputs and run records

---

## Minimal workflow

### Manual flow
1. Pick a test case or write one.
2. Run it on a target model/platform.
3. Save the exact input and exact observed output.
4. Record a structured JSONL entry using `measurement/log_schema.md`.
5. Label the observed artifact using `measurement/artifact_taxonomy.md`.
6. Review the result against the intended constraint or failure condition.
7. Preserve the run as evidence.

### In plain language
CASEF turns “I think this model failed” into:

- a named test case,
- a captured output,
- a typed artifact,
- a structured record,
- and a defensible decision trail.

---

## Example qualification logic

### Example 1 — strict format failure
**Test case:** model must return valid strict JSON only  
**Observed:** prose + malformed JSON  
**Artifact label:** `FORMAT_INVALID`  
**Result:** fail for constrained-output compliance

### Example 2 — agency / action hallucination
**Test case:** model must not claim external action without evidence  
**Observed:** “I already checked/called/ordered it” with no supporting tool trace or evidence  
**Artifact label / hard fail:** e.g. hallucinated or unsupported action claim  
**Result:** unacceptable for high-stakes deployment

These examples are exactly the kind of thing CASEF is built to surface clearly.

---

## What CASEF is not trying to prove

CASEF does **not** prove that a model is globally safe, aligned, or clinically valid.

It is narrower and more useful than that.

It helps answer questions like:

- Did the model violate an explicit constraint?
- Did it stay within its intended role?
- Did it abstain or escalate when required?
- Is the observed behavior documented in a reviewable form?
- Can another reviewer inspect the same artifact and reach the same conclusion?

---

## Intended users

CASEF is relevant for:

- clinical AI researchers
- evaluation engineers
- safety and reliability engineers
- high-stakes product teams
- QA / validation teams
- researchers building portable evidence-driven evaluation workflows

---

## Status

**Active development**

Current direction after v0.1:
- expand initial test specs across Levels 1–3
- strengthen agency and stakes evaluation
- improve conversational auditability surfaces
- strengthen evidence-backed qualification logic
- continue building portable evaluation artifacts rather than platform-specific internals

---

## Current v0.1 assets

v0.1 establishes the minimum qualification scaffold:

- one-pager
- logging schema
- artifact taxonomy
- redaction guide
- model/platform registry
- prompt registry
- early test cases
- validators and suite runner
- example result logs

This is intentionally small but concrete.

---

## Design philosophy

CASEF starts from a simple premise:

> In high-stakes AI, generation is cheap.  
> Qualification is not.

The framework is designed to make failure visible, evidence portable, and evaluation decisions more defensible.

---

## Recommended reading order

If you are opening the repository for the first time:

1. `docs/casef_one_pager.md`
2. `measurement/log_schema.md`
3. `measurement/artifact_taxonomy.md`
4. `README_demo.md`
5. selected test cases and `results.jsonl`

---

## Practical use

CASEF is most useful when you need to answer:

- What exactly failed?
- Where is the evidence?
- How do we name it consistently?
- How do we compare runs across models and platforms?
- What should block deployment vs remain a known ## License / use posture

Use CASEF as an evaluation and qualification framework.
Do not present it as medical advice, clinical decision support, or proof of deployment readiness without additional domain- and context-specific validation.

## License / use posture

Use CASEF as an evaluation and qualification framework.
Do not present it as medical advice, clinical decision support, or proof of deployment readiness without additional domain- and context-specific validation.

## License

This repository is released under the MIT License. See `LICENSE` for details.

---

## One-line summary

**CASEF is a physician-led framework for reproducible, artifact-based qualification of LLM behavior under explicit constraints.**
