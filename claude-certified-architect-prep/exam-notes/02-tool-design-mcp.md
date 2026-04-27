# 02 — Tool design (MCP)

## What “good” looks like

- **Narrow** tools (do one thing well)
- **Stable** contracts (inputs/outputs don’t drift)
- **Structured** outputs (machine-checkable)
- **Typed** errors (recoverable vs unrecoverable)
- **Idempotent** where possible (safe retries)

## Contract template

- **Name**:
- **Purpose**:
- **Inputs**:
  - required:
  - optional:
- **Output**:
- **Errors**:
  - validation:
  - auth:
  - not_found:
  - transient:
- **Examples**:

## Reliability notes

- Validate inputs early.
- Prefer small, composable tools.
- Return enough metadata for retries and debugging.

