# Claude in CI/CD Flashcards

## Q: Why is CI/CD different from using Claude in the IDE?

A: **No interactive user**; jobs must be **non-interactive**, outputs often need to be **machine-parseable**, and failures need clear **exit behavior**.

## Q: CI job hangs waiting for input — what is the usual fix?

A: Use **headless / non-interactive** invocation (per `claude --help` for your version) so nothing waits on stdin or prompts.

## Q: Why use `--output-format json` (or equivalent) in pipelines?

A: So the **next step** (`jq`, scripts, PR comments) can **parse** results reliably.

## Q: Why validate agent JSON output with a schema?

A: Catches **shape drift** before posting PR comments or failing builds; avoids broken automation.

## Q: What is a safe pattern for PR comments from agent output?

A: Parse JSON in a **small audited script**; do **not** execute model text as shell.

## Q: What is an “independent review” context in CI?

A: A **clean checkout** or isolated diff so the model sees **consistent** code, not local editor noise.

## Q: How can you reduce false positives from CI agent reviews?

A: **Explicit criteria** in prompts, **severity** thresholds, allowlists for noisy paths, optional second-stage filter.

## Q: What is multi-pass review in CI?

A: Run **cheap** checks first (lint, tests), then **heavier** agent passes only when worth the cost.

## Q: Should agent output always block merges?

A: Usually **advisory** unless policy explicitly gates on it; define **blocking vs non-blocking** severities.

## Q: Single one-shot validation in CI — plan mode or direct?

A: Often **direct execution** is enough; avoid extra planning overhead for trivial checks.

## Q: Where do structured errors fit in CI agent steps?

A: Tools should return **`isError`, `errorCategory`, `isRetryable`** so the pipeline can retry, fail, or escalate (`03-structured-errors-and-reliability.md`).

## Q: Before relying on CLI flags in YAML, what should you do?

A: Confirm flags against **`claude --help`** in the **same** image/version as production CI.
