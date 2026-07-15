# CASEF Repository Audit — Scientific / Technical / Product

- **Date:** 2026-07-15
- **Scope:** full repository at commit `8ffd4a2` (main), all 37 tracked files (~1,490 lines), GitHub metadata (issues, PRs), and live execution of the runner.
- **Type:** independent scientific–technical–product audit. Findings are graded using CASEF's own severity spirit: **S1** (workflow/consistency), **S2** (undermines trust or usability), **S3** (undermines the framework's core claim).

---

## 1. Executive summary

CASEF's core idea is genuinely strong and differentiated: treating LLM safety as a **qualification problem** (spec → measurement → gates → evidence) rather than a benchmark score is the right frame for clinical deployment, and the severity-first gate path (`risk_map.md` → `gates.md`) is a defensible governance design. The test-case writing quality (especially `TC-L3-EVAL-*` and `CA-*`) is above average for an early-stage safety repo.

However, the repository currently **fails its own standard**. A framework whose thesis is "reproducible, inspectable, defensible evidence" ships a flagship evidence pack that is internally inconsistent, a logging schema with three competing field vocabularies, and a runner that — when actually executed — validates **zero** of its own demo outputs. This is the single most important strategic risk: every inconsistency in an evidence-first repo is a counterexample to the pitch.

**Verdict by CASEF's own logic:** the *concept* passes; the *current artifact set* would be `FAIL` at its own qualification gate. All of this is fixable in one focused consistency release (proposed as v0.7 below).

| Dimension | Grade | One-line assessment |
|---|---|---|
| Scientific methodology | ⚠️ Early (4/10) | Good constructs, no measurement rigor yet (n=1, no IRR, contamination unaddressed) |
| Technical implementation | ❌ Weak (3/10) | Runner broken against own demo, schema drift, no tests/CI |
| Internal consistency | ❌ Weak (3/10) | Violates its own ADR and canon rules in multiple places |
| Product positioning | ✅ Promising (6/10) | Clear niche and honest non-goals; overclaims "clinical-grade"; no related-work anchoring |
| Documentation | ⚠️ Mixed (5/10) | Strong individual docs; README corrupted/duplicated; broken cross-references |

---

## 2. Scientific / methodological audit

### 2.1 What is scientifically sound

- **Severity before likelihood** (`measurement/risk_map.md`) is explicitly justified and honest ("we avoid simulating precision"). This is good epistemic hygiene.
- **Observed-behavior-only scope** (no telemetry claims, `redaction/guide.md` provenance rule) is a correct and unusual discipline.
- **Separation of evidence inputs from decision engine** (`docs/gates.md` v0.3: `hard_fail_type`/`rater_label` are classification inputs, S3 mapping is the single gate) avoids the common multi-gate contradiction problem.
- The **eval-awareness module** (v0.5) and **conversational audit layer** (v0.6) target real, under-measured phenomena (frame sensitivity, test-frame downgrade, auditable uncertainty).

### 2.2 Methodological gaps

| # | Severity | Finding |
|---|---|---|
| SCI-1 | **S3** | **n = 1 sampling.** Every demo run is a single generation per model/platform. LLM outputs are stochastic; a single sample supports no claim, including a FAIL claim ("this model CAN fail the contract" is defensible; the framework nowhere says this is the only claim being made). No repeat-count, temperature, or sampling policy is specified anywhere. |
| SCI-2 | **S3** | **No inter-rater reliability protocol.** The v0.3 gate path is explicitly rater-driven ("severity assignment is primarily manual"), yet there is no double-rating requirement, no adjudication rule, no agreement statistic (Cohen's κ), and no calibration set. For a framework marketed to governance teams, unmeasured rater subjectivity in the *authoritative* decision path is the largest scientific hole. |
| SCI-3 | **S2** | **Test-set contamination is unaddressed.** All prompts are published verbatim ("copy-paste"). Future model versions will likely train on this repo; pass rates will inflate and regression comparisons (a stated core feature: "rerun suites on model updates and compare deltas") become confounded. There is no held-out / private-variant policy. |
| SCI-4 | **S2** | **Construct validity of TC-L3-EVAL-02.** Asking a model to *role-play* "answer as if this were a benchmark" vs "as if real user" measures instructed frame-following, not genuine evaluation awareness. A model can pass while still behaving differently under *actual* covert evaluation. TC-L3-EVAL-03 ("only a test" pressure) is stronger; TC-L3-EVAL-02 should be re-labeled as a *frame-instruction sensitivity* probe, not an eval-awareness detector. |
| SCI-5 | **S2** | **Format constraints confound safety constructs.** Most L3 safety tests impose 90–120-word budgets and exact-count structures. A model can hold a perfect safety boundary and land at 121 words. The specs correctly demote this to S1 — but binding `constraint_pass` into the same PASS/FAIL as safety behavior (docs/gates ADR: FAIL if `constraint_pass=false`) means the headline metric mixes two constructs. Report format compliance and safety compliance as separate outcomes. |
| SCI-6 | **S2** | **Severity is assigned per failure-mode class, not per instance.** `risk_map.md` maps e.g. UNSAFE_REASSURANCE → S2 globally, while the definitions say severity depends on "a reasonable intended-use scenario." A context-of-use modifier (which the table's "Default decision" column half-implies) is not formalized. |
| SCI-7 | **S1** | **No statistical treatment at all** — no pass-rate definitions, no confidence intervals, no minimum-run counts for a gate decision. Even a one-paragraph "decision requires ≥k consistent observations" rule would materially strengthen the qualification claim. |
| SCI-8 | **S1** | **Level 2 (multi-turn stress & semantic invariance) has zero assets** despite being one of three core levels in the one-pager and arguably the most clinically distinctive one (table → JSON → leaflet invariance). |

### 2.3 Scientific quick wins

1. Add `measurement/protocol.md`: samples per test (n ≥ 5), fresh-session rule, temperature policy, and the exact claim a FAIL supports.
2. Add a 2-rater + adjudication rule and report κ once ≥ 30 rated runs exist.
3. Add a contamination policy: public spec + private paraphrase variants per test_id.
4. Split `constraint_pass` into `format_pass` and `safety_pass` in the next schema revision.

---

## 3. Technical audit

### 3.1 Executed verification (not just code reading)

Running the runner from the repo root reproduces two defects:

- `python3 measurement/run_suite.py` → **`ModuleNotFoundError: No module named 'measurement'`** (absolute import inside the package, no `__init__.py`, no packaging). Only `python3 -m measurement.run_suite` works, and no doc says so.
- `python3 -m measurement.run_suite` → writes 3 rows, **all with `"notes": "no validators wired for this test_id"`**. The dispatch matches on `f.stem == "TC-L1-JSON-01"`, but the actual demo files are `TC-L1-JSON-01_claude_opus-4.5.txt` etc., so the stems never match. **The only executable component in the repo validates none of its own demo evidence.**

### 3.2 Code findings (`measurement/run_suite.py`, `measurement/validators.py`)

| # | Severity | Finding |
|---|---|---|
| TEC-1 | **S3** | Dispatch bug above (`run_suite.py:25,41`): filename stems include model suffixes → validators never run. Also `test_id` is polluted with the model suffix in the emitted rows. |
| TEC-2 | **S2** | `run_suite.py:56` opens `results.jsonl` in append mode with no run_id/dedup — every invocation silently duplicates rows into the *committed example evidence file*. An evidence log that self-corrupts on rerun contradicts the reproducibility goal. |
| TEC-3 | **S2** | Emitted rows hardcode `level: "L1"`, `model/platform: "UNKNOWN"`, omit `run_id`, `date_utc`, `input`, `output` — i.e., the runner's own output does not conform to `measurement/log_schema.md` (which marks those fields required). |
| TEC-4 | **S1** | `validators.py:16–25` — `v_json_keys_exact` parses the JSON twice (calls `v_json_parse`, then `json.loads` again). |
| TEC-5 | **S1** | `wordcount` (`validators.py:27`) counts `\b\w+\b`: hyphenated words count as 2, numbers and underscores count as words. For tests whose PASS hinges on 90–120 words, "word" needs a written definition, or borderline runs are rater-dependent. |
| TEC-6 | **S1** | `v_no_bullets` misses numbered lists (`1.`, `1)`) and en-dash bullets (`–`), which TC-L1-COUNT-02's "no bullet lists" plausibly intends to forbid. |
| TEC-7 | **S1** | The validator `V_WORDCOUNT(answer) <= 40` required by `spec/tests/level1/TC-L1-JSON-01.md` is not implemented anywhere; `spec/prompt_registry.md` references validator names (`V_JSON_PARSE`, `V_WORDCOUNT`) that only loosely correspond to actual function names. |
| TEC-8 | **S2** | **Zero tests, zero CI.** No pytest for validators, no GitHub Actions, no schema validation of `results.jsonl`. For a repo whose product *is* validation, a CI job that validates the repo's own example artifacts is the highest-leverage 50 lines available. |
| TEC-9 | **S1** | No `.gitignore` (running the suite leaves `measurement/__pycache__/` dirty), no `requirements.txt`/`pyproject.toml` (stdlib-only today — fine, but state it), no `__init__.py`. |

### 3.3 Schema and vocabulary drift (violates the repo's own ADR)

`docs/decisions/ADR_2026-01-06_ids-and-fields.md` standardizes canonical fields precisely because "we observed drift." The drift is still present:

| Field concept | log_schema.md (v0.1) | gates ADR canon | results.jsonl | run_suite.py |
|---|---|---|---|---|
| artifact labels | `artifact_types` | `artifacts` | **both**, with *different meanings* (`artifacts` = taxonomy labels, `artifact_types` = free-text sub-detail) | `artifacts` |
| timestamp | `date_utc` (required) | — | `timestamp_utc` | absent |
| run identity | `run_id` unique per run (required) | — | same `run_id` shared across 3 runs | absent |
| no-hard-fail value | `hard_fail_type: null` | `"NONE"` | `null` | `"NONE"` |
| input/output capture | required | — | absent | absent |

Additional label drift:

- **`RENDERING_LEAK`** (`artifact_taxonomy.md`) vs **`RENDERING_LEAKAGE`** (`risk_map.md`, `results.jsonl`) — same artifact, two names; grep-based tooling will silently miss one. (S2)
- `SAFETY_ESCALATION_MISS` gates via `rater_label`, `HALLUCINATED_ACTION` et al. gate via `hard_fail_type` — two parallel enums feed the same gate with no stated reason; `log_schema.md`'s `hard_fail_type` enum omits `SAFETY_ESCALATION_MISS` entirely. (S2)
- `TC-L3-AUTONOMY-02` lists `FORMAT_INVALID` under "Hard-fail mapping" although it is an artifact/S1 label, not a member of the hard-fail enum. (S1)

### 3.4 Spec/evidence integrity

| # | Severity | Finding |
|---|---|---|
| INT-1 | **S3** | **One test_id, two different tests.** `spec/tests/level1/TC-L1-JSON-01.md` requires keys `["answer","confidence"]`; `examples/demo_run/inputs/TC-L1-JSON-01.txt` requires keys `["patient_age","chief_complaint","risk_level","next_step"]`. This directly violates the prompt-registry canon rule ("every test has a stable test_id"), and means the runner's wired key-check would fail *correct* outputs of the demo prompt. |
| INT-2 | **S2** | **Manifest ↔ files mismatch.** `run_manifest.md` lists output files `gpt-5.2-thinking__chatgpt__TC-L1-JSON-01.txt` (double-underscore convention); actual files are `TC-L1-JSON-01_chatgpt_gpt-5.2-thinking.txt`. None of the three referenced filenames exist. |
| INT-3 | **S2** | **Model identity inconsistency inside the evidence pack.** Manifest says "Gemini 3 Flash"; `results.jsonl` says "Gemini 2.0 Flash"; the file name says only `gemini_flash`. The Claude row's `model_reported` (`claude-opus-4-5-20250514`) is dated 2025-05-14 for a model marketed as Opus 4.5 — an unverifiable/likely-incorrect ID recorded as observed fact. The Claude row's `timestamp_utc` (2026-01-10) also contradicts its `run_id` date (2026-01-06). For a framework whose registry exists to prevent exactly this, the flagship example undermines the pitch. |
| INT-4 | **S2** | `spec/prompt_registry.md` is stale: it lists 3 tests; the repo contains 12 (`TC-L3-AUTONOMY-01..03`, `TC-L3-EVAL-01..03`, `CA-01..03` are unregistered). The registry's own canon rule is violated by 9 of 12 tests. |
| INT-5 | **S1** | Tests live in two roots with no stated rule: `spec/tests/level1/` vs `framework/level3/...` — plus `spec/model_registry.md` under a third convention. Pick one tree (e.g., `spec/tests/<level>/<module>/`). |
| INT-6 | **S1** | `docs/decisions/ADR_2026-01-06_ids-and-fields.md` is two concatenated documents (the full "Gates (v0.2)" doc followed by the ADR) — a copy-paste accident that makes the ADR hard to cite. |
| INT-7 | **S1** | Version fragmentation with no map: log schema v0.1, tests/runner v0.2, gates/risk v0.3, rater labels v0.4, eval labels v0.5, audit layer v0.6 — and no CHANGELOG or `suite_state.md` (the one-pager promises `/spec/suite_state.md`; it does not exist). |

### 3.5 Documentation defects

- **README.md:279–289** — corrupted section: a sentence breaks off mid-line into `## License / use posture`, and the whole "License / use posture" block is then **duplicated verbatim**. This is on the repo's front page. (S2)
- **README "Repository contents"** lists `prompt_registry.md`, `run_suite.py`, `run_manifest.md`, `README_demo.md` without their real paths (`spec/`, `measurement/`, `examples/demo_run/`). (S1)
- **One-pager** references three files that don't exist: `/spec/suite_state.md`, `/redaction/redaction_guide.md` (actual: `redaction/guide.md`), `/docs/not_jailbreak.md`. (S1)
- `examples/conversation_audit/RRADME_demo.md` — filename typo (**RRADME** → README). (S1)
- README is ~300 lines with heavy repetition (the "what CASEF is/is not" idea is restated ≥5 times); the strongest content (workflow, qualification examples) is buried mid-file. (S1)

### 3.6 GitHub-side state

- 0 issues (open or closed), 2 PRs (both self-merged within ~30 seconds of opening — i.e., branches used as a changelog, no review process), no CI workflows, no releases/tags, no branch protection signals, no CONTRIBUTING/CITATION.cff/issue templates. Repo description is good and matches the one-liner.

---

## 4. Product audit

### 4.1 Strengths

- **Real niche.** "Qualification evidence for governance teams" is distinct from leaderboard benchmarks (HELM, HealthBench-style) and from red-teaming suites. The buyer persona (clinical governance, QA/validation, insurers) is plausible and underserved.
- **Honest non-goals** ("not an AI doctor", "not proof of deployment readiness", no jailbreak content) are repeated consistently and reduce regulatory/comms risk.
- **The conversational audit layer is the most productizable asset**: `operator_accept_reject` + `provenance_trace` + `replay_reference` maps directly onto human-in-the-loop clinical workflows that governance teams already understand.
- Physician-led authorship is a credible differentiator *if* backed by rigor (see risks).

### 4.2 Product risks and gaps

| # | Severity | Finding |
|---|---|---|
| PRO-1 | **S3** | **"Clinical-grade" is an overclaim today.** The term implies validation evidence (IRR, protocolized measurement, traceable versions) that the repo does not yet have — and sophisticated buyers (hospital QMS, regulatory-adjacent teams) will check. Recommend "clinical-context" or "designed toward clinical-grade" until §2.3 items land. |
| PRO-2 | **S2** | **No regulatory anchoring.** For the stated buyers, mapping CASEF concepts onto ISO 14971 (risk_map ≈ hazard analysis), IEC 62304 / IEC 82304, FDA GMLP, and EU AI Act high-risk QMS duties would be the single strongest adoption lever — and it is entirely absent. |
| PRO-3 | **S2** | **No related-work positioning.** No comparison to HealthBench, MedSafetyBench, Inspect/OpenAI Evals, or agent-safety suites. Without it, researchers can't cite CASEF and buyers can't tell whether it duplicates something they already have. One honest table would do. |
| PRO-4 | **S2** | **No working quickstart.** The advertised 7-step manual workflow is fine, but the automated path is broken (TEC-1), so a motivated evaluator's first 10 minutes end in a stack trace or a "no validators wired" log. First-run experience is the product for an OSS framework. |
| PRO-5 | **S1** | **No adoption surface**: no CONTRIBUTING, no CITATION.cff, no issue templates, no "add your own test case" tutorial, no example of a completed *governance decision memo* built from an evidence pack (the actual end deliverable buyers would pay attention to). |
| PRO-6 | **S1** | Public-prompt contamination (SCI-3) is also a product problem: the framework's regression story ("compare deltas across model updates") is its main recurring-use case, and contamination erodes exactly that. |

### 4.2b Strategic finding: relevance drift (single-turn vs agentic)

| # | Severity | Finding |
|---|---|---|
| PRO-0 | **S3** | **CASEF qualifies what a model *says*; the 2026 deployment risk is what an agent *does* across a multi-step trajectory.** Every test in the repo assumes a single-turn textual output. Even the agency/autonomy modules (TC-L3-AGENCY-01, TC-L3-AUTONOMY-03) probe *claims about* actions, not actual tool-using behavior. The gap is not broken code or missing code — it is that the unit of evaluation is one layer below where the industry's evaluation problem now lives. |

The framework's own vocabulary is already pointed at this (HALLUCINATED_ACTION, `final_action_trace`, `provenance_trace`, `replay_reference`) — extending the unit of evaluation from *response* to *trajectory* is an evolution of CASEF, not a new repo. Concretely: an **Agent Trajectory Audit Schema** — steps, tool calls, and evidence bound to each state-changing action — plus one hand-authored example trajectory and a schema validator. Scope guard: one scenario, hand-written; no agent-framework integration in the first cycle.

### 4.3 Recommended roadmap

**v0.7 — "Consistency release" (1–2 days of focused work, highest ROI):**
1. Single canonical schema as a machine-readable JSON Schema (`measurement/log_schema.json`); regenerate `results.jsonl` to conform; delete one of `artifacts`/`artifact_types`.
2. Fix `run_suite.py`: parse `TESTID_platform_model.txt` filenames, wire validators, write conformant rows, make it runnable as a script, don't append duplicates.
3. Unify labels (`RENDERING_LEAK` vs `RENDERING_LEAKAGE`), merge the `hard_fail_type`/`rater_label` gate enums or document the split.
4. Repair README (corrupted/duplicated sections, wrong paths), fix the ADR file, rename `RRADME_demo.md`, complete `prompt_registry.md` for all 12 tests, add CHANGELOG + `spec/suite_state.md`.
5. Resolve the TC-L1-JSON-01 dual-prompt conflict (new test_id for the clinical variant, e.g. `TC-L1-JSON-02`).
6. Add `.gitignore`, pytest for validators, and a CI workflow that (a) runs the tests, (b) validates `results.jsonl` against the JSON Schema. CI validating the repo's own evidence *is* the product demo.

**v0.8 — measurement rigor:** sampling protocol (n≥5), rater/IRR protocol, format-vs-safety outcome split, contamination policy.

**v0.9 — content:** Level 2 module (semantic invariance is the most clinically distinctive idea in the repo and currently has zero assets), **first agentic qualification scenario** (trajectory schema + one worked example + validator, per §4.2b — the strategic move), related-work table, regulatory mapping doc.

---

## 5. Consolidated findings register

| ID | Severity | Area | Summary |
|---|---|---|---|
| TEC-1 | S3 | Code | Runner validates none of its own demo outputs (filename dispatch bug) |
| INT-1 | S3 | Spec | TC-L1-JSON-01 exists as two different tests under one test_id |
| SCI-1 | S3 | Method | n=1 sampling, no sampling protocol |
| SCI-2 | S3 | Method | Rater-driven authoritative gate with no IRR/adjudication protocol |
| PRO-0 | S3 | Product | Relevance drift: single-turn unit of evaluation; no agentic/trajectory scenarios |
| PRO-1 | S3 | Product | "Clinical-grade" claim unsupported by current rigor |
| TEC-2 | S2 | Code | Runner append-corrupts committed evidence file on rerun |
| TEC-3 | S2 | Code | Runner output violates required log schema |
| TEC-8 | S2 | Infra | No tests, no CI, no schema validation of evidence |
| §3.3 | S2 | Schema | `artifacts` vs `artifact_types`, `date_utc` vs `timestamp_utc`, `null` vs `"NONE"`, dual gate enums, RENDERING_LEAK(AGE) |
| INT-2 | S2 | Evidence | Manifest references non-existent output filenames |
| INT-3 | S2 | Evidence | Model names/IDs/timestamps inconsistent inside flagship evidence pack |
| INT-4 | S2 | Spec | Prompt registry missing 9 of 12 tests |
| SCI-3 | S2 | Method | No contamination / held-out policy for public prompts |
| SCI-4 | S2 | Method | TC-L3-EVAL-02 measures instructed role-play, not eval awareness |
| SCI-5 | S2 | Method | Format constraints confound safety outcomes in one PASS/FAIL |
| SCI-6 | S2 | Method | Severity fixed per failure class, not per context of use |
| PRO-2..4 | S2 | Product | No regulatory mapping, no related work, broken first-run path |
| README | S2 | Docs | Corrupted + duplicated license section on front page |
| TEC-4..7, TEC-9 | S1 | Code | Validator precision issues, missing validator, hygiene files |
| INT-5..7 | S1 | Structure | Dual test roots, concatenated ADR, version fragmentation, dead links |
| SCI-7..8 | S1 | Method | No statistical rules; Level 2 empty |
| PRO-5..6 | S1 | Product | No adoption surface; contamination hurts regression story |

---

## 6. Closing assessment

CASEF is a good idea documented with above-average conceptual clarity and below-average execution discipline. Nothing found here is fatal; almost everything in §3 is mechanical to fix, and §2 requires writing protocols rather than building software. The framework's credibility model is unusual: **the repo itself is the evidence pack**. Once the repo passes its own gates — schema-valid evidence, a runner that reproduces the committed results, one canonical vocabulary — the same artifacts that are currently liabilities become the strongest possible demo.

*This audit is an evaluation artifact, not medical or regulatory advice, and follows the repository's own convention: observed > claimed.*
