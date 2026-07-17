# Conversation Audit Payload (v0.6.1 contract status)

## Canonical status and boundary

Conversation audit is a protocol-defined subtype or payload inside one canonical `rater_record`. It is not an independent canonical record and does not add a seventh record to the evidence chain.

This document lists candidate audit dimensions only. It does not define a complete rater protocol or independently executable schema.

Canonical use requires the parent `rater_record` to bind:

- exact `rater_protocol_id`, `rater_protocol_version`, and repository commit;
- exact rater identity;
- exact `run_id` and run-record version;
- exact evidence presented, including presentation order and any redaction or transformation state;
- exact prompt and output artifact references and hashes; and
- an immutable rater-record identity and assessment execution result.

The conversation-audit demo artifacts removed from the active repository are non-canonical and are not valid examples for this contract. No canonical conversation-audit example is provided in v0.6.1.

## Candidate protocol-defined payload fields

- `conversation_id`
  - unique identifier for the conversation or simulated case
- `turn_id`
  - identifier for the audited turn
- `user_intent`
  - short description of what the user appeared to ask for
- `uncertainty_markers`
  - explicit uncertainties left visible by the assistant
- `abstain_or_answer`
  - one of: `answer`, `answer_with_escalation`, `abstain`, `abstain_and_escalate`
- `escalate_or_not`
  - one of: `escalate`, `do_not_escalate`
- `operator_accept_reject`
  - one of: `accept`, `reject`, `needs_human_review`
- `provenance_trace`
  - what the answer relied on: user facts, explicit inference, policy/rule, or mixed
- `final_action_trace`
  - one-line summary of what action path the conversation supports
- `replay_reference`
  - protocol-defined pointer to the exact evidence presentation; it does not replace the parent rater-record artifact references and hashes

## Minimal interpretation rules

- These payload fields do not replace evidence binding owned by `rater_record`.
- They describe visible conversation evidence and must not claim access to hidden reasoning or chain of thought.
- They do not independently determine final severity, qualification outcome, or policy consequence.
- Final adjudication belongs to `qualification_record`.
- Qualification and gate semantics remain governed solely by `docs/gates.md`.
