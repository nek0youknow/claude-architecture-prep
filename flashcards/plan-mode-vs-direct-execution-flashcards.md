# Plan Mode vs Direct Execution Flashcards

## Q: What is plan mode?

A: The agent **plans steps first**, then executes them in order (often multi-tool or multi-stage).

## Q: What is direct execution?

A: The model **acts immediately** with minimal upfront planning — best for simple, obvious tasks.

## Q: When should you prefer plan mode?

A: **Multi-step** work, **several tools/agents**, unclear ordering, or need a **visible plan** for audit or approval.

## Q: When should you prefer direct execution?

A: **Single** clear action (one lookup, one command), low risk, and planning would add little value.

## Q: Example task suited to plan mode?

A: Cross-domain **research + synthesis** (e.g. gather sources, analyze, then one structured report).

## Q: Example task suited to direct execution?

A: **Order status** check with one `lookup_order` call.

## Q: Main downside of plan mode?

A: **Higher latency/tokens**; risk of over-planning on simple tasks.

## Q: Main downside of direct execution?

A: On **risky** flows, easy to **skip steps** or call tools in the wrong order — fix with **hooks/prerequisites**, not only “plan more.”

## Q: Exam: many steps and tools — what pattern?

A: **Plan mode** or explicit coordinator steps.

## Q: Exam: single validation check — what pattern?

A: Often **direct execution**; avoid unnecessary full planning overhead.

## Q: Does plan mode replace programmatic guards for refunds/identity?

A: **No.** Hard rules still need **code enforcement** (`05-hooks-and-escalation.md`).

## Q: User asks for order status — wrong tool to call?

A: Use **`lookup_order`** (or equivalent), **not** `process_refund` or other unrelated tools.
