# Prompt Engineering Flashcards

## Q: Why are vague prompts bad?

A: They lack specificity and constraints, so the model fills gaps unpredictably — wrong format, wrong scope, or plausible-but-wrong content.

## Q: How can you improve vague prompts?

A: Add **role, goal, context, constraints**, and an **explicit output contract** (format, fields, enums). Prefer **examples** when shape matters.

## Q: What is explicit criteria in prompt engineering?

A: Clear, testable rules: what to include, what to skip, how to handle missing data, and what counts as an error vs unknown.

## Q: What are few-shot examples?

A: Input/output pairs (or short demonstrations) that show the **exact structure** and tone you want.

## Q: How do few-shot examples help the model?

A: They reduce ambiguity about **format** and **edge-case handling**; the model mimics the pattern instead of guessing.

## Q: What is false positive reduction in prompt engineering?

A: Tightening instructions so the model **does not flag** or **does not escalate** issues that are not real under your definition (noise control).

## Q: What is severity criteria?

A: A rubric that maps findings to levels (e.g. blocking vs advisory) so outputs are **prioritized** consistently.

## Q: What is the best way to define specific report/skip criteria?

A: Use **explicit rules** plus **examples** of include/skip/borderline; avoid hand-wavy words like “important” without definitions.

## Q: How do explicit criteria reduce false positives?

A: They narrow what counts as a hit; the model has less freedom to over-report irrelevant patterns.

## Q: Why does “be conservative” often fail as sole guidance?

A: It is not operational — the model may be **silent** on real issues or **noisy** on trivia. Replace with **defined conditions**, thresholds, and examples.

## Q: How does prompt engineering relate to structured output?

A: You specify **schema-friendly** fields (JSON keys, enums); downstream code **validates** and can **retry** with parse errors as feedback.

## Q: When is prompt engineering not enough for safety?

A: For **money, identity, permissions, privacy, irreversible actions** — add **programmatic guards/hooks** (`05-hooks-and-escalation.md`), not only better wording.

## Q: What should you specify for refusal/uncertainty?

A: What to do when **information is missing**: ask a targeted question, refuse, or escalate — instead of guessing.
