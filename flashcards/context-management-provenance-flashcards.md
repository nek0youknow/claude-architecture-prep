# Context Management and Provenance Flashcards

## Q: What is "lost in the middle"?

A: LLMs tend to recall the **beginning** and **end** of context better than the **middle**; critical facts there may be overlooked.

## Q: How do you mitigate lost in the middle?

A: Reordering important facts, **summaries**, **retrieval**, **scratchpads**, and **structured state** (not burying constraints only in mid-history).

## Q: What is context trimming?

A: Removing less important context to cut tokens/latency while keeping what matters.

## Q: What should context trimming preserve?

A: **Constraints**, **decisions**, **unresolved issues**, and **case facts**.

## Q: What are case facts?

A: Persistent, high-signal information about the case that must not be dropped (e.g. customer verified, refund approved).

## Q: Example of a case fact?

A: Customer identity verified.

## Q: What is a scratchpad?

A: Internal **working memory** for intermediate reasoning, extracted facts, plans, and open questions.

## Q: Why use scratchpads?

A: **Continuity** and multi-step reasoning without re-deriving everything from raw chat each turn.

## Q: What does `/compact` do?

A: Compresses long context to save tokens while trying to preserve decisions, constraints, and key facts (Claude Code).

## Q: What is claim–source mapping?

A: Linking each important **claim** to the **source** (document, page, chunk) that supports it.

## Q: Why is provenance important?

A: **Traceability** and catching unsupported or **hallucinated** claims.

## Q: What is conflict annotation?

A: When credible sources **disagree**, surfacing **both** with attribution and why they might differ — not collapsing to one number silently.

## Q: What should the system do when two credible sources conflict?

A: Preserve **both** with attribution and explain possible reasons for discrepancy; do not arbitrarily pick one.

## Q: What is wrong when sources conflict?

A: **Arbitrarily choosing** one source or stating a single fact with false certainty.

## Q: When should systems escalate to human review?

A: Low confidence, conflicting evidence, or high-risk domains (legal, medical, financial).

## Q: Why avoid relying only on raw chat history in production?

A: Long-context **memory is unreliable**; important state should live in **structured** stores, not only the transcript.

## Q: Provenance vs citation — difference?

A: **Provenance** is internal claim→source traceability; **citation** is how you present sources to the user.

## Q: Context window vs scratchpad / case state?

A: Context is **ephemeral input**; scratchpad and case state are **persistent working memory** for the workflow.
