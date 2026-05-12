# 10 — Context management and provenance

This topic is about **reliability**, **traceability**, **hallucination prevention**, **source integrity**, and **context survival** — where real AI systems diverge from a simple chatbot.

## Core problem

Large systems juggle long conversations, many documents, multiple agents, huge contexts, conflicting sources, retrieval, scratchpads, and summaries.

Constraints:

- The **context window is finite** (cost, latency, limits).
- Even with a large window, **attention is imperfect** — facts can be lost or deprioritized.

You need **context management**, **provenance**, **attribution**, **conflict handling**, and **human review** where appropriate.

## Lost in the middle

Models often attend more strongly to the **beginning** and **end** of context than the **middle**. Important facts buried mid-history can be **forgotten or contradicted**.

Example: “Customer is premium enterprise” appears early; 50 large messages; later “issue resolved”; more bulk — the model may suggest upgrading to premium because the key fact was **not salient**.

### Mitigations

1. **Context reordering** — place critical constraints and facts near the **start** or **end**, not buried in the middle.
2. **Summaries** — compress raw history into a **summary state** with explicit invariants.
3. **Scratchpad** — persist important intermediate facts outside the raw transcript.
4. **Retrieval** — inject only **relevant** chunks instead of full history.

## Context trimming

When context is too large, **remove less important** material.

### Bad trimming

Dropping something like “customer identity already verified” → model asks for verification again.

### Good trimming

Remove: greetings, irrelevant chatter, duplicated explanations, stale irrelevant detail.

Preserve: **constraints**, **case facts**, **decisions**, **approvals**, **unresolved issues**.

## Case facts

**Case facts** are persistent, high-signal state that must not be lost across a long thread.

Examples:

- customer identity verified
- refund approved / denied
- legal escalation started
- priority tier (e.g. enterprise)
- invoice disputed
- root cause identified

### Architecture

Do not rely only on raw chat. Prefer **structured case state**, e.g.:

```json
{
  "customerVerified": true,
  "refundApproved": false,
  "priorityLevel": "enterprise",
  "caseStatus": "investigating"
}
```

## Scratchpad

A **scratchpad** is **working memory** for the agent: intermediate reasoning, extracted facts, plans, unresolved questions, temporary notes.

Example content:

- Invoice total = 1000; line items sum = 900; possible missing tax; re-check page 2.

Why it helps: **continuity**, multi-step reasoning, consistency, less repeated recomputation.

### Scratchpad vs final answer

- **Scratchpad**: internal / agent-facing.
- **Final answer**: clean, user-facing output (do not dump raw scratchpad to end users unless intended).

## `/compact` (Claude Code)

When a session is huge, **compact** compresses context to reduce tokens while **preserving** what matters: decisions, constraints, unresolved issues, architecture, important facts.

**Bad compact**: drops “auth already implemented”, “migration done”, “approval already given” → duplicate work.

## Claim–source mapping (provenance)

Every **material claim** should be traceable to a **source**.

- **Provenance**: internal traceability (which claim came from which document/chunk/page).
- **Citation**: user-facing phrasing (“According to McKinsey …”) — related but not the same layer.

Example:

```json
{
  "claim": "AI market size is $12B",
  "source": {
    "document": "McKinsey AI Report 2025",
    "page": 18
  }
}
```

Production rule: avoid **high-confidence claims without attribution**.

## Conflict annotation

When **credible sources disagree**, do **not** silently pick one.

Wrong: “Market size is $15B” with no mention of the other estimate.

Right: state both with attribution and uncertainty (definitions, methodology, scope may differ).

Why it matters: the system should **preserve uncertainty**, not fake certainty.

## Human review

Escalate when:

- conflicting legal interpretations
- medical uncertainty
- low-confidence extraction
- high-risk financial actions
- contradictory evidence and policy is unclear

Good pattern: “Confidence low due to conflicting sources — escalating for human review.”  
Bad pattern: silent guess.

## Multi-agent provenance

Research agent outputs “market = $12B” but synthesis loses **where** that came from → **lost attribution**.

Fix: pass **structured** claim + source objects between agents, not only free text.

## Reliability hierarchy (rough)

1. Pure LLM memory of long chat — weakest.
2. Retrieval + summaries — better.
3. Structured state + provenance — better.
4. Validation + provenance + human review where needed — strongest for high-stakes.

## RAG-style pipeline sketch

1. Retrieve documents  
2. Extract claims  
3. Attach sources  
4. Detect conflicts  
5. Synthesize with conflict annotation where needed  
6. Escalate uncertain / high-risk conflicts  
7. Generate answer with citations where appropriate  

## Common exam traps

| Question shape | Wrong | Right |
|----------------|--------|--------|
| Two credible sources conflict | Pick one | Preserve both + attribution + discrepancy note |
| Very long, expensive context | Hope the model remembers | Trim + summarize + structured state |
| Model forgets verification | Repeat in prompt only | Persist **case facts** |
| Long workflow needs reasoning continuity | Rely on chat | **Scratchpad** |
| Constraint buried in middle | — | **Lost in the middle** — reorder / summarize / retrieve |
| Lower tokens, keep essentials | Drop random chunks | **`/compact`** or summarization with explicit invariants |

## One-line summary

Do not rely only on raw chat history. Use **structured state**, **summaries**, **retrieval**, **scratchpads**, **claim–source mapping**, **conflict annotation**, and **human review** for high-risk uncertainty.

## Related notes

- `05-context-reliability.md` — context sources and reliability tactics  
- `09-validation-retry-batch-processing.md` — validation after extraction  
- `08-structured-output.md` — structured payloads for state and provenance  
- Flashcards: `flashcards/context-management-provenance-flashcards.md`
