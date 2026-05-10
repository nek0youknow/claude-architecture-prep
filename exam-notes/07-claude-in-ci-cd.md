# 07 — Claude Code in CI/CD

## Why CI/CD is different from the IDE

In CI you typically have:

- **No human** at the keyboard to answer prompts.
- **Ephemeral** environments; failures must be **machine-readable** (exit codes, JSON, JUnit, SARIF).
- **Secrets and permissions** must be explicit (tokens, allowlists, read-only scopes).

So the goals are: **non-interactive runs**, **structured output**, and **predictable failure modes**.

## CLI patterns (conceptual)

Exact flag names can change by **Claude Code / CLI version** — always run `claude --help` in your pipeline image. The **ideas** below are what exam and real pipelines care about.

### Non-interactive / “don’t wait for stdin”

- Use the mechanism your CLI provides for **headless** or **print** mode so jobs **do not hang** waiting for input.
- Common pattern: pass the **prompt or command** as an argument or stdin once, with **no follow-up questions** (or use flags that force non-interactive behavior).

**Exam answer shape**: “CI hung waiting for input → use **non-interactive / print / headless** flags (e.g. `-p` or equivalent per docs).”

### Structured output for downstream steps

- **`--output-format json`** (or similar): emit JSON so the next step can parse findings (severity, file paths, suggested fixes).
- Pair with a **schema** or **json-schema** validation when you need strict shape for `jq`, custom gates, or PR comment scripts.

Example pattern (illustrative):

```bash
claude <your-subcommand-or-prompt> --output-format json > claude-result.json
```

### Validation

- **`--json-schema`** (when supported): validate output against a schema **before** posting to PRs or failing the build — avoids broken automation.

## Typical CI/CD flow

1. **Trigger**: commit, PR, schedule, or release tag.
2. **Build & test**: unit tests, lint, typecheck (existing jobs).
3. **Agent step** (optional): code review summary, security triage, doc drift check, migration risk note.
4. **Publish**: artifact (JSON/HTML), PR comment, or status check.
5. **Policy**: treat agent output as **advisory** unless you explicitly gate merges on it.

## Example: agent output → PR comment

```bash
claude <headless-invocation> --output-format json > result.json
node scripts/add-pr-comment.js result.json
```

Keep `add-pr-comment.js` (or equivalent) **small and audited** — it should not execute arbitrary model text as shell.

## Additional practice areas

### Independent review context

Run reviews in a **clean checkout** or **isolated worktree** so the model sees a **consistent diff** and not local editor noise.

### PR comments

- Summarize **actionable** items; link to lines or files.
- Separate **blocking** vs **nonsuggestions** if your team uses merge rules.

### False positives

- Tighten prompts with **explicit include/exclude criteria** (see `flashcards/prompt-engineering-flashcards.md` and `04-prompt-engineering-structured-output.md`).
- Post-process: severity thresholds, allowlists for known noisy paths.

### Multi-pass review

Chain **fast** checks first (lint, tests), then **heavier** agent passes only if cheap gates pass — saves cost and reduces noise.

## Mini checks

| Symptom | Likely fix |
|---------|------------|
| CI job **hangs** waiting for input | Non-interactive / headless invocation; no interactive prompts |
| Need to **parse** results in the next job | Structured output (e.g. JSON) + schema validation |
| Single **one-shot** validation | Often **direct execution** is enough; avoid multi-step planning overhead |
| Agent comments are **noisy** | Stricter criteria, severity rubric, second-stage filter |

## Link to other notes

- Workflows (commands, rules, skills): `03-claude-code-workflows.md`
- Plan vs direct: `06-plan-mode-vs-direct-execution.md`
- Reliability and errors: `03-structured-errors-and-reliability.md`
- Flashcards: `flashcards/claude-in-ci-cd-flashcards.md`
