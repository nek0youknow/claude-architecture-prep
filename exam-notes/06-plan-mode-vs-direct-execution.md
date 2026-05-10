# 06 — Plan mode vs direct execution

## Core idea

- **Plan mode**: the agent (or coordinator) **outlines steps first**, then executes them in order. Good when the task is **multi-step**, **multi-tool**, or needs **explicit sequencing**.
- **Direct execution**: the model **acts immediately** with minimal upfront planning. Good when the path is **obvious** and **one or two tool calls** suffice.

Exam heuristic: complexity and **risk** drive the choice — not “always plan” or “always jump.”

## Plan mode

**Plan mode** means the model first **plans all (or most) steps**, then carries them out **sequentially** (or delegates to subagents). Use it for **complex** work: several stages, several sources, or unclear ordering.

### Example task (research / synthesis)

- **Goal**: Summarize how AI shows up in film, music, and literature.
- **Steps**:
  1. Gather sources per domain (search / docs).
  2. Extract claims and citations.
  3. Synthesize one coherent report with a shared structure.

### Coordinator-style sketch

```text
Coordinator:
  1. Search AI in music     → Web search / research agent
  2. Search AI in film      → Web search / research agent
  3. Analyze gathered docs    → Analysis pass
  4. Synthesize final report → Single structured output
```

### When to prefer plan mode

- Multiple steps or **dependencies** (B before C).
- Several **tools** or **subagents** (search + DB + codegen).
- Need a **visible plan** for debugging, review, or human approval mid-flight.
- High **blast radius** — you want checkpoints before irreversible actions.

### Tradeoffs

- **Pros**: clearer reasoning trace, easier to audit, fewer wrong orderings.
- **Cons**: extra latency and tokens; over-planning on simple tasks.

## Direct execution

**Direct execution** means **minimal planning** — go straight to the right tool or answer when the task is **simple and unambiguous**.

### Example task (single lookup)

- **Goal**: Check order status.
- **Steps**: Call `lookup_order` (or equivalent) with the order id.

### Example (correct tool)

```text
User: Check my order #123
Agent: Call lookup_order(order_id="123")
```

Do **not** use a refund tool for a status check — match **intent** to **tool**.

### When to prefer direct execution

- One clear action: “get X”, “run lint”, “answer from this snippet.”
- Low risk and **no** prerequisite chain (or prerequisites already satisfied).
- Speed matters and planning adds little value.

### Tradeoffs

- **Pros**: fast, cheap, simple.
- **Cons**: easy to **skip verification** or **wrong tool order** on risky flows — then you need **guards/hooks** (see `05-hooks-and-escalation.md`), not just “plan harder.”

## Comparison

| Criterion | Plan mode | Direct execution |
|-----------|-----------|------------------|
| Task shape | Complex, multi-step, multi-source | Simple, single-step or obvious path |
| Example | Cross-domain research report | Order status lookup |
| Tools / agents | Often several | Usually one (or very few) |
| Typical cost / latency | Higher upfront | Lower |
| Risky domains (money, identity) | Plan **and** enforce prerequisites in code | Still use **programmatic gates** even if execution feels “direct” |

## Exam-style mini checks

- **Many steps and tools?** → Prefer **plan mode** (or explicit coordinator steps).
- **Single validation or lookup?** → Prefer **direct execution**; avoid over-planning.
- **Agent skips verification before a refund?** → Fix with **prerequisite / hook**, not only plan mode or prompt.

## Link to other notes

- Agent loop and decomposition: `01-agentic-loop.md`
- When prompts are not enough: `05-hooks-and-escalation.md`
- Structured output contracts: `04-prompt-engineering-structured-output.md`
- Flashcards: `flashcards/plan-mode-vs-direct-execution-flashcards.md`
