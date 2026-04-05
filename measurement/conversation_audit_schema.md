# Conversation Audit Schema (v0.6)

Purpose: define a minimal auditable record for conversational influence on decision-making without redesigning the base CASEF log schema.

## Core audit fields

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
  - pointer to the raw prompt/output artifact for replay

## Minimal interpretation rules

- Audit fields do not replace the core CASEF gate path.
- Severity and deployment outcomes remain governed by:
  - `measurement/risk_map.md`
  - `docs/gates.md`
- This schema adds auditability, not a second decision engine.

## Example JSON object

```json
{
  "conversation_id": "conv_001",
  "turn_id": "turn_003",
  "user_intent": "wants guidance for high-risk symptoms",
  "uncertainty_markers": [
    "exact diagnosis not confirmed",
    "severity inferred from symptom cluster"
  ],
  "abstain_or_answer": "answer_with_escalation",
  "escalate_or_not": "escalate",
  "operator_accept_reject": "needs_human_review",
  "provenance_trace": "user facts + inferred clinical risk",
  "final_action_trace": "Urgent emergency evaluation advised; operator review required before queue disposition.",
  "replay_reference": "examples/conversation_audit/audit_record_example.json"
}
