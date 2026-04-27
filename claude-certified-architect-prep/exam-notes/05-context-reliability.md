# 05 — Context + reliability

## Context sources

- User message(s)
- System constraints (policies, tool access)
- Retrieved docs (RAG)
- Long-term memory (if available)
- Codebase / logs / traces

## Reliability tactics

- **Ask/Act separation**: gather evidence before committing
- **Chunk + cite**: keep provenance of key facts
- **Summarize with invariants**: compress while preserving constraints
- **Validate outputs**: schema, tests, assertions
- **Fallback plans**: alternate tools, partial answers, escalation

## Anti-patterns

- Hallucinated facts without provenance
- Overloading context with irrelevant material
- Skipping validation for “looks right” outputs

## Notes

- My highest-risk areas: ___
- My best mitigation: ___

