# Hooks and Escalation Flashcards

## Q: Why is a prompt not enough for critical business rules?

A: Prompts are probabilistic. Critical rules need deterministic programmatic enforcement (guards/hooks).

## Q: What is the “exam formula” for reliability?

A: Prompt guides behavior; programmatic enforcement guarantees behavior.

## Q: What is a programmatic prerequisite (gate)?

A: A code-level condition that must be true before a dangerous tool can run.

## Q: Example of a prerequisite in a refund flow?

A: Block `process_refund` unless `context.verified_customer_id` exists.

## Q: What is a hook in agent workflows?

A: A function that runs at a specific point to intercept/modify the workflow (e.g., before/after tool use).

## Q: What is tool call interception (BeforeToolUse)?

A: Checking/blocking/modifying a tool call **before** it executes.

## Q: Agent sometimes skips customer verification before refund. Best fix?

A: Add a prerequisite/hook that blocks refund/order tools until verification succeeds.

## Q: Agent sometimes refunds above $500. Best fix?

A: Intercept `process_refund` and block amounts above $500 with a structured business error + escalate.

## Q: What is PostToolUse?

A: A hook that runs **after** tool execution to normalize, redact, or transform tool results before the model uses them.

## Q: When should you use PostToolUse?

A: When tool outputs are inconsistent (dates/status codes), too verbose, or include sensitive fields.

## Q: User explicitly asks for a human. What should the agent do?

A: Escalate immediately.

## Q: Policy is ambiguous or silent (policy gap). What should the agent do?

A: Escalate to a human (don’t invent policy).

## Q: Customer is angry. Always escalate?

A: No. Sentiment is not a reliable trigger; escalate only by explicit request, risk criteria, or inability to progress.

## Q: What is a “structured handoff”?

A: A concise escalation payload with customer ID, issue, actions attempted, tool results summary, policy reason, and recommended action.

## Q: Prompt vs guard — what’s the difference?

A: Prompt tells the model what to do; guard/hook prevents forbidden actions regardless of model behavior.

