# CASEF v0.6.1 Context-of-Use contract

## 1. Purpose and qualification requirement

A Context of Use defines the exact use boundary to which a CASEF qualification decision applies. No canonical qualification may be authorized without one bounded, immutable, versioned Context of Use.

The Context of Use is a referenced specification, not a seventh canonical evidence record. A `qualification_record` binds its exact `context_id`, `context_version`, content reference, and content hash.

This document defines the minimum contract only. It is not a clinical product dossier, regulatory template, or Context-of-Use instance.

## 2. Minimum field dictionary

All fields below are required for a complete Context of Use.

| Field | Requirement | Meaning | Controlled vocabulary | Versioning effect | Qualification relevance |
|---|---|---|---|---|---|
| `context_id` | Required | Stable identity of one bounded intended-use construct | Repository-governed identifier; filenames are not authoritative | A materially different intended-use construct requires a new ID | Prevents a decision from being transferred to a different use |
| `context_version` | Required | Exact immutable version of the Context of Use | Repository-approved version identifier | Any substantive field change requires a new version | Binds qualification to the reviewed boundary |
| `domain` | Required | Bounded application domain in which the use occurs | No universal v0.6.1 enum; use a precise domain term or governed reference | Change requires a new version; a different domain normally requires a new ID | Establishes the domain assumptions and claim boundary |
| `intended_users` | Required | Explicit user and operator roles for the use | Bounded list of roles | Change requires a new version; a materially different user population may require a new ID | Determines who may rely on or act on the system |
| `intended_task` | Required | Exact task or decision-support purpose | Bounded structured statement | Material change normally requires a new ID; clarification requires a new version | Defines what the qualification does and does not cover |
| `use_mode` | Required | Mode in which the evaluated system is intended to be used | `RESEARCH_EVALUATION`, `ADVISORY`, `OPERATIONAL` | Any change requires a new version and ordinarily a new decision | Separates research, advice, and operational use |
| `stakes` | Required | Material consequences reasonably associated with the intended use | Bounded structured statement; no universal v0.6.1 severity enum | Any change requires a new version; material escalation may require a new ID | Determines applicable evidence, review, and gate-policy requirements |
| `action_mode` | Required | Maximum state-changing authority allowed in the use | `NO_STATE_CHANGE`, `HUMAN_AUTHORIZED_STATE_CHANGE`, `AUTONOMOUS_STATE_CHANGE` | Any change requires a new version and ordinarily a new decision | Defines whether output can directly change external state |
| `permitted_actions` | Required | Explicit actions within the evaluated authority boundary | Explicit bounded list | Any addition or removal requires a new version | Limits what a positive qualification could permit |
| `prohibited_actions` | Required | Explicit actions outside the evaluated authority boundary | Explicit bounded list | Any addition or removal requires a new version | Prevents silence from being interpreted as authorization |
| `human_oversight` | Required | Oversight role, intervention point, review obligation, override authority, and escalation boundary | Structured description; absence must be explicit and must not use a sentinel string | Any change requires a new version | Determines where human authorization or review is required |
| `deployment_boundary` | Required | Exact interface, operating environment, and in-scope and out-of-scope deployment boundary | Structured bounded description | Any boundary change requires a new version; material expansion may require a new ID | Prevents transfer of a decision to an unassessed environment |

## 3. Controlled values

`use_mode` uses exactly:

- `RESEARCH_EVALUATION` — controlled research or evaluation activity; not operational authorization;
- `ADVISORY` — output may inform a human decision but does not itself authorize state change; and
- `OPERATIONAL` — output participates in a live operational workflow within the stated action and oversight boundary.

`action_mode` uses exactly:

- `NO_STATE_CHANGE` — the system cannot directly change external state;
- `HUMAN_AUTHORIZED_STATE_CHANGE` — an identified human authority must approve state change; and
- `AUTONOMOUS_STATE_CHANGE` — the system may change external state within the explicitly bounded authority.

These vocabularies describe use conditions. They do not determine qualification outcome or policy consequence by themselves.

## 4. Immutability and versioning

Once referenced by a qualification decision, a Context-of-Use version is immutable. A typographical or metadata correction creates a new version and an explicit supersession relation; it does not overwrite the referenced version.

A change that materially alters the intended task, domain, intended users, use mode, stakes, action authority, oversight model, or deployment boundary creates a new `context_id` when it defines a different intended-use construct. Compatible clarification that preserves the intended-use construct still requires a new `context_version`.

An earlier qualification does not transfer automatically to a later Context-of-Use version.

## 5. Missing-value rules

Required Context-of-Use fields cannot be null, absent, empty, or replaced by sentinel strings such as `"NONE"`, `"N/A"`, or `"UNKNOWN"`.

If a required boundary fact is unresolved, the Context of Use is incomplete and no canonical qualification record or qualification outcome may be created. `REVIEW_REQUIRED` applies only after a complete bounded Context of Use exists and a downstream evidence, assessment, adjudication, or authority requirement remains unresolved.

Optional future extension fields may use actual null or field absence only when their contract permits it. Absence of human oversight must be represented explicitly in structured content, not hidden behind a sentinel value.

## 6. Qualification and claim boundary

A qualification record must reference the exact immutable Context of Use and apply only the gate policy valid for that context. Qualification evidence remains scoped to the exact evaluated model or exposed alias, product, interface, date, test contract, runs, and decision inputs.

Evidence from an individual run or bounded evaluation cannot support:

- a general model-safety conclusion;
- a provider-wide or company-wide conclusion;
- a product-wide conclusion;
- a permanent conclusion about a moving model alias; or
- authorization outside the stated Context of Use.

Provider, product, and model identity may be recorded as provenance. They do not expand the decision boundary.

## 7. Current v0.6.1 implementation status

This document defines a documentation-level contract. It does not create a Context-of-Use instance, JSON Schema, clinical dossier, gate rule, qualification decision, or executable validation.
