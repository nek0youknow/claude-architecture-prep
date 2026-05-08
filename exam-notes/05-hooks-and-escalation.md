# 05 — Hooks, guards, and escalation

## Core idea

Prompt instructions are useful but **not sufficient** for critical business rules.

- Prompt **guides** model behavior (probabilistic).
- Programmatic enforcement **guarantees** behavior (deterministic).

RU shortcut:

- Prompt направляет модель.
- Кодовая проверка гарантирует правило.

## Problem pattern (exam-favorite)

Agent sometimes violates a hard rule (money, identity, privacy, permissions, irreversible actions).

Bad fix: “make the prompt stronger”.

Best fix: add a **guard / prerequisite / hook** that blocks unsafe tool calls.

## Programmatic prerequisite (gate)

A code-level requirement that must be satisfied before a dangerous tool can run.

Example rule:

- `process_refund` must not run unless `verified_customer_id` exists in context.

Example tool-result error returned by runtime (not the model):

```json
{
  "isError": true,
  "errorCategory": "business",
  "isRetryable": false,
  "message": "Customer verification is required before processing a refund.",
  "recommendedAction": "call_get_customer_first"
}
```

Where to implement prerequisites:

- Tool handler (inside `process_refund`)
- Middleware/runtime (generic `beforeToolCall`)
- Agent SDK hook (intercept tool calls)

## Hook

A function that runs at a specific point in the agent workflow.

Exam-relevant hook types:

- **BeforeToolUse / tool call interception**: block/redirect before tool executes
- **PostToolUse**: normalize/redact/transform tool output before model uses it

## Tool call interception (BeforeToolUse)

Use it to enforce:

- prerequisites (identity verified before refund)
- thresholds (refund amount \(\le\) $500)
- permissions (staff-only operations)
- confirmations (delete account only after explicit confirm)
- escalation routing (policy exceptions)

Example: block refunds above $500 even if the prompt says not to:

```json
{
  "isError": true,
  "errorCategory": "business",
  "isRetryable": false,
  "message": "Refund amount exceeds the automatic approval limit of $500.",
  "recommendedAction": "escalate_to_human"
}
```

## PostToolUse (normalize + redact)

Runs after a tool executes, but before the model consumes the result.

Use it when:

- different tools return inconsistent formats (dates/statuses)
- tool output is noisy (40 fields, debug traces)
- tool output contains sensitive data (payment tokens, internal notes)

Example: normalize dates to a single field:

```json
{ "created_at_iso": "2026-05-01T10:00:00Z" }
```

## Prompt vs hook/guard (exam rule)

| Situation | Prompt enough? | Need hook/guard? |
|---|---:|---:|
| tone / style | yes | no |
| “be concise” | yes | no |
| format preference | usually | no |
| identity verification before refund | no | yes |
| refund threshold enforcement | no | yes |
| do not expose private data | no | yes |
| delete account only after confirmation | no | yes |
| compliance requirement | no | yes |

## Escalation (handoff to human / safer workflow)

Escalate when:

- user explicitly asks for a human
- policy is ambiguous/silent (policy gap)
- business exception is required (e.g., over threshold)
- identity cannot be verified
- multiple customer matches cannot be safely disambiguated
- tool returns `permission`/`business` error requiring staff approval
- repeated failures prevent meaningful progress

Important nuance (common trap):

- Negative sentiment alone (angry user) is **not** a reliable trigger.

## Structured handoff (exam “good escalation”)

Don’t escalate with “please help”. Provide a structured summary that lets a human act quickly:

```json
{
  "customer_id": "cust_777",
  "customer_verified": true,
  "issue_type": "refund_request",
  "order_id": "ORD-123",
  "refund_amount": 800,
  "root_cause": "Customer reports damaged item",
  "actions_attempted": [
    "Verified customer identity",
    "Looked up order",
    "Checked refund eligibility",
    "Attempted refund but amount exceeds automatic approval limit"
  ],
  "policy_reason": "Refund amount exceeds $500 threshold",
  "recommended_action": "Manager approval required",
  "tool_results_summary": {
    "order_status": "delivered",
    "refund_eligible": true,
    "refund_blocked_reason": "amount_limit"
  }
}
```

## Exam answer templates

- If the agent violates a hard business rule: **add a prerequisite / intercept tool calls** (don’t just strengthen the prompt).
- If tool outputs are inconsistent: **PostToolUse normalization**.
- If escalations lack context: **structured handoff**.

