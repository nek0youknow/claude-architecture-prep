# Structured Errors Flashcards

## Q: What is isError?

A: A flag indicating that a tool call failed.

## Q: What is a transient error?

A: A temporary failure like timeout, service unavailable, or network issue.

## Q: Should transient errors be retryable?

A: Usually yes.

## Q: What is a validation error?

A: Invalid or missing input.

## Q: Should validation errors be retried?

A: Not with the same input. Fix the input or ask the user.

## Q: What is a permission error?

A: Access denied or insufficient authorization.

## Q: Should permission errors be retried?

A: No, request access or escalate.

## Q: What is a business error?

A: A policy or business rule prevents the operation.

## Q: Should business errors be retried?

A: No, explain policy or escalate if needed.

## Q: What are partial results?

A: Useful data returned before a tool or subagent failed.

## Q: Why include partial results?

A: So the coordinator can continue, retry only missing parts, or mark coverage gaps.

## Q: Valid empty result vs error?

A: Empty result means successful query with no matches. Error means the tool failed.

## Q: Timeout search agent — what should be returned?

A: Structured error context with failure type, attempted query, partial results, and alternatives.

## Q: Why is "Operation failed" bad?

A: It does not tell the agent whether to retry, ask the user, escalate, or stop.

## Q: What should structured error include?

A: errorCategory, isRetryable, message, attempted input, partial results, and recommended action.

