# 04 — Prompt engineering + structured output (lecture)

## Core idea

A **prompt** is the instruction and context you give the model. Good prompt engineering **reduces ambiguity**, **locks output shape** when needed, and **defines behavior under uncertainty**. It **steers** the model — it does **not** replace **code-level guards** for money, identity, or irreversible actions (see `05-hooks-and-escalation.md`).

## Building blocks (use together)

1. **Role** — who the model should act as (e.g. senior reviewer, support agent).
2. **Goal** — the single outcome you want.
3. **Context** — facts, policies, snippets, or retrieved docs the model must respect.
4. **Constraints** — length, tone, language, what **not** to do, tools available.
5. **Output contract** — format (JSON fields, markdown sections), enums, required vs optional keys.
6. **Uncertainty policy** — what to do when information is missing (ask, refuse, escalate, partial answer with caveats).

### Quick template (copy-paste skeleton)

**Task**:

**Context**:

**Constraints**:

**Output (JSON or sections)**:

**If information is missing**:

**Validation** (how a program checks the output):

**Retry rule** (if validation fails, what to fix and resend):

## Vague vs explicit prompts

| Vague | More explicit |
|-------|----------------|
| “Review this PR.” | “List only **blocking** issues: security, correctness, breaking API. One bullet per issue with file:line. If none, return `issues: []`.” |
| “Be careful.” | “Flag only if condition X holds; otherwise mark `severity: none`.” |
| “Summarize the doc.” | “3 bullets, max 15 words each; third bullet = single open risk.” |

**Why vague prompts fail**: the model **fills gaps** with plausible defaults — wrong format, wrong scope, or confident hallucination.

## Few-shot examples

**Few-shot** = show **input → output** pairs (or short demonstrations) so the model copies **structure** and **edge-case handling**.

Use few-shot when:

- Output shape is easy to get wrong (tables, nested JSON, strict sections).
- You want consistent handling of **borderline** cases (include one “skip” and one “flag” example).

Avoid dumping huge example sets; **3 tight examples** often beat 20 noisy ones.

## Explicit criteria, severity, and false positives

- **Explicit criteria** — testable rules: what counts as a finding, what to **skip**, and what “unknown” means.
- **Severity rubric** — e.g. `blocking` vs `advisory` vs `info`; tie each level to **conditions**.
- **False positive reduction** — narrow definitions (“only if …”) and **negative examples** (“do not flag when …”).

**Why “be conservative” alone fails**: it is not operational. The model may stay silent on real bugs or spam noise. Replace with **conditions + examples + severity**.

## Structured output

When downstream code or another agent must **parse** the answer:

- Prefer **JSON** with **small, meaningful** keys.
- Use **enums** for allowed values (`severity`, `status`).
- Keep nesting shallow; avoid redundant prose inside JSON strings.

### Validation and retry loop

1. Model emits JSON.
2. Your code **validates** against a schema or required keys.
3. On failure, return **parse/validation errors** to the model and **retry** once or twice with a short “fix these fields” message.

This pattern improves reliability more than asking for “valid JSON” in one shot without validation.

## Refusal, hedging, and escalation (prompt layer)

Specify **explicitly**:

- When to **say you don’t know** vs guess.
- When to **ask the user** one targeted question vs proceed.
- When to **escalate** (policy gap, missing permission, ambiguous identity).

This pairs with **structured errors** from tools (`03-structured-errors-and-reliability.md`).

## Chain-of-thought and reasoning (light touch)

For complex reasoning, you can ask for **short internal steps** or **sections** (e.g. “Assumptions” then “Conclusion”). Keep it **bounded** (step count, length) so you do not explode context. For **user-facing** answers, you can ask for **final answer only** after reasoning, or separate “scratchpad” from “customer reply” in multi-field JSON.

## Limits of prompt engineering (exam-critical)

Prompts **do not guarantee** compliance with hard business rules. For refunds over a threshold, identity before payout, PII exposure, deletes, payments — use **prerequisites, hooks, interception** (`05-hooks-and-escalation.md`).

## Exam heuristics

| Symptom | Strong lever |
|---------|----------------|
| Wrong format / schema | Explicit output contract + few-shot + validate/retry |
| Too many noisy findings | Explicit include/skip + severity rubric + examples |
| Model guesses when data missing | Uncertainty policy + structured “unknown” state |
| Hard rule still violated | Programmatic guard, not only prompt tuning |

## Drill

Pair this note with `flashcards/prompt-engineering-flashcards.md`.

## Related notes

- Plan vs direct execution: `06-plan-mode-vs-direct-execution.md`
- Hooks and escalation: `05-hooks-and-escalation.md`
- Structured errors: `03-structured-errors-and-reliability.md`
- Claude in CI (prompts for automation): `07-claude-in-ci-cd.md`
