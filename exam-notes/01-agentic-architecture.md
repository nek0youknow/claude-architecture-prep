# 01 — Agentic architecture

## Core components

- **Goal/spec**: what “done” means
- **Context**: inputs, retrieved docs, memory
- **Reasoning loop**: plan → act → observe → reflect
- **Tools**: bounded actions with contracts
- **State**: what persists across steps (and why)
- **Guardrails**: policy, safety, budget, latency constraints

## Design principles

- Prefer **simple loops** over complex orchestration until needed
- Make tool contracts **explicit and testable**
- Treat failures as **first-class states** (retry, fallback, escalate)
- Record **provenance** for important outputs

## Common failure modes

- Missing/incorrect context
- Tool misuse (wrong params, wrong tool)
- Non-deterministic outputs without validation
- Overly long context leading to dilution
- Silent partial failure (timeouts, truncation)

## Checklist

- What is the “source of truth”?
- Where is state stored?
- How do we validate outputs?
- What do we do on tool failure?
- How do we keep costs/latency bounded?

