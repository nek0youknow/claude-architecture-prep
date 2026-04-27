# 04 — Prompt engineering + structured output

## Prompt patterns

- **Role + goal + constraints**
- **Examples** (few-shot) when format matters
- **Explicit output contract** (schema, keys, enums)
- **Refusal/uncertainty handling** (what to do when missing info)

## Structured output guidance

- Prefer JSON with a schema.
- Keep fields small and meaningful.
- Use enums for “allowed values”.
- Validate and **retry** with error feedback if invalid.

## Quick template

**Task**:

**Context**:

**Output (JSON)**:

**Validation**:

**Retry rule**:

