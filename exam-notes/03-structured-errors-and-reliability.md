# Structured Errors and Reliability

## Core idea

Agents should not receive generic errors like "Operation failed".

They need structured error context to decide whether to retry, ask the user, escalate, continue with partial results, or stop.

## Key fields

- isError
- errorCategory
- isRetryable
- message
- attemptedOperation
- attemptedInput
- partialResults
- attemptedRecovery
- alternatives
- recommendedAction

## Error categories

| Category | Meaning | Retry? |
|---|---|---|
| transient | temporary failure: timeout, service unavailable | usually yes |
| validation | invalid or missing input | usually no |
| permission | access denied | no |
| business | policy/business rule blocks operation | no |

## Valid empty result vs error

Valid empty result:

```json
{
  "isError": false,
  "results": []
}
```

This means the query succeeded but found nothing.

Access failure:

```json
{
  "isError": true,
  "errorCategory": "permission",
  "isRetryable": false
}
```

This means the tool failed.

Do not treat access failure as no results.

## Partial results

If a tool or subagent fails after finding some data, return partial results.

This allows the coordinator to:

- retry
- use alternative tools
- continue with caveats
- mark coverage gaps

## Exam rule

If a search subagent times out, return structured error context:

- failure type
- attempted query
- partial results
- attempted recovery
- alternatives
- coverage gaps

Do not return only "search unavailable".

