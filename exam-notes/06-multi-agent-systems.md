# 06 — Multi-agent systems

A **multi-agent system** is a setup where a **coordinator agent** delegates parts of a task to multiple **specialized subagents**, then aggregates the results into a final answer.

## 1) What is a multi-agent system?

**Single agent:**

User → Agent → Tools → Final answer

**Multi-agent system:**

User
 ↓
Coordinator Agent
 ├── Subagent 1: Web Search
 ├── Subagent 2: Document Analysis
 ├── Subagent 3: Code Analysis
 └── Subagent 4: Synthesis
 ↓
Final answer

Core idea: the coordinator **does not do everything**. It **decomposes** the task, delegates work, and merges outputs.

## 2) Why multi-agent systems exist

A single agent can handle small tasks:

- Summarize this short file.

But large tasks are harder for one agent:

- Research AI impact on creative industries using web sources, PDFs, and internal reports. Compare findings and create a cited report.

Common failure pressures in “one-agent-does-everything” mode:

- **Context bloat** → details get lost
- **Too many tool calls** in one loop
- Role confusion (search + analysis + synthesis + formatting simultaneously)
- Harder quality control (coverage, correctness, provenance)
- Harder recovery from partial failures (timeouts, truncated results)

## 3) Coordinator

The **coordinator** is the “PM / team lead” for agents. It owns the end-to-end result.

Responsibilities:

- task decomposition
- subagent selection
- explicit context passing
- error handling
- result aggregation
- coverage checking
- final answer production

### Example: “Research AI impact on creative industries”

Coordinator decides:

- this is broad research
- domains to cover: music, film, publishing/writing, visual arts, gaming, advertising/design
- sources + attribution must be preserved
- results must be comparable across domains

Coordinator delegates:

- Web Search Agent → AI in music
- Web Search Agent → AI in film
- Document Agent → publishing reports (PDFs)
- Synthesis Agent → combine findings

### Anti-pattern: coordinator does everything

Bad: coordinator searches, reads, analyzes, writes the report itself.

Why it fails:

- too much context
- too many tool calls in one place
- attention dilution
- higher chance of missing whole areas
- worse error recovery

## 4) Subagent

A **subagent** is a specialized agent with a narrow scope and an expected output format.

Examples:

| Subagent | Job |
| --- | --- |
| Web Search Agent | find web sources |
| Document Analysis Agent | read/analyze PDFs/docs |
| Code Analysis Agent | explore a codebase |
| Test Generation Agent | write tests |
| Synthesis Agent | combine findings |
| Report Agent | format final deliverable |
| Verification Agent | verify claims and provenance |

Subagent design constraints:

- specific role
- role-appropriate tools only
- specific prompt
- specific output format
- limited scope

## 5) Hub-and-spoke architecture

Hub-and-spoke means: **all subagent communication is routed through the coordinator**.

        Web Search Agent
              ↑
              |
Document Agent ← Coordinator → Synthesis Agent
              |
              ↓
        Verification Agent

Bad: subagents talk directly to each other (harder to debug, track errors, and control context).

Exam phrasing:

- Q: “How should subagent communication be routed for observability and consistent error handling?”
- A: **Route all subagent communication through the coordinator.**

## 6) Task tool (spawning subagents)

The coordinator uses a **Task tool** (conceptually) to spawn/invoke subagents.

Exam point:

- If the coordinator cannot spawn subagents, the likely missing configuration is: `allowedTools` must include **`Task`**.

Conceptually:

- Task(web_search_agent, "Research AI in music. Return findings with sources.")

Pseudo-JSON:

```json
{
  "type": "tool_use",
  "name": "Task",
  "input": {
    "agent": "web_search_agent",
    "prompt": "Research AI in music industry. Return findings with source URLs and dates."
  }
}
```

## 7) Must memorize

**Subagents do not automatically inherit parent context.**

Meaning: subagents do not see the coordinator’s history, tool results, or “what we already found” unless you explicitly paste it into their prompt.

### Prompt quality: bad vs good

Bad:

- “Create a final synthesis from previous research.”

Why it fails:

- the subagent may not have “previous research” in its context
- it may guess, hallucinate, or produce generic output
- provenance can be lost

Good (explicit context passing):

- You are the synthesis agent.
- Task: produce a report about AI impact on creative industries.
- Use ONLY the findings below (insert them).
- Requirements: preserve attribution, identify patterns, identify gaps, do not invent claims.

## 8) Explicit context passing

Explicit context passing = the coordinator **manually includes** what the subagent needs.

What to pass (practical checklist):

1) Task goal
2) Scope boundaries
3) Prior findings (verbatim, preferably structured)
4) Provenance metadata (URL, date, document name, page number, who found it, confidence)
5) Output format/schema
6) Constraints (no new browsing, no unsupported claims, keep attribution)

## 9) Parallel subagents

Parallel subagents = the coordinator launches multiple independent subagent tasks concurrently.

Sequential (slow when independent):

- search music → wait
- search film → wait
- search publishing → wait

Parallel (lower latency):

- Task(web_search_agent: music)
- Task(web_search_agent: film)
- Task(document_agent: publishing)

Exam phrasing:

- Q: “How to reduce latency when multiple independent research subtasks are needed?”
- A: **Spawn parallel subagents by emitting multiple Task calls in a single coordinator response.**

## 10) Iterative refinement

Iterative refinement = the coordinator does not stop after the first synthesis; it **checks quality/coverage**, then **runs more targeted subagents** and re-synthesizes.

Example:

1) User: “Research AI impact on creative industries.”
2) Coordinator spawns: music, film, publishing
3) Synthesis returns a draft report
4) Coordinator checks coverage:
   - Covered: music, film, publishing
   - Missing: gaming, visual arts, writing
5) Coordinator spawns additional subagents for missing areas
6) Coordinator re-runs synthesis with the expanded findings

Why it matters: without iterative refinement, the final answer can be incomplete even if every subagent “worked.”

Exam diagnosis:

- Scenario: final report covers only visual arts; music/writing/film are missing; subagents ran successfully; logs show the coordinator decomposed into only digital art/graphic design/photography.
- Root cause: **Coordinator task decomposition was too narrow.**

## 11) Task decomposition

Task decomposition = how the coordinator splits a broad request into smaller subproblems.

Good decomposition for “creative industries”:

- music
- film
- publishing/writing
- visual arts
- gaming
- advertising/design

Bad decomposition:

- digital art
- graphic design
- photography

Why bad: it only covers the visual slice and misses major domains (music, film, writing, gaming, publishing).

How the coordinator should think:

- What are the major parts of the topic?
- Which parts need different tools?
- Which subtasks are independent (parallelizable)?
- Which subtasks depend on prior outputs?
- What quality criteria must the final answer satisfy (citations, comparisons, gaps)?

## 12) Role-specific tools

Each subagent should have a **scoped toolset** aligned to its role.

Bad: Synthesis Agent has tools like:

- search_web
- load_document
- write_file
- process_refund
- query_database
- send_email

Why bad:

- worse tool selection
- higher misuse risk
- harder safety auditing
- harder debugging
- more unexpected behavior

Good:

- Web Search Agent: `search_web`, `fetch_url`
- Document Agent: `load_document`, `extract_text`
- Synthesis Agent: maybe `verify_fact`; **no broad search**
- Coordinator: `Task` + aggregation/planning helpers

Exam rule:

- Give each subagent only the tools relevant to its role.

## 13) Structured output from subagents

Subagents should return **structured output**, not long messy prose.

Bad:

- “I found some articles. AI affects music and film. There are copyright issues…”

Good (example shape):

```json
{
  "topic": "AI in music",
  "findings": [
    {
      "claim": "AI tools are used for composition and production assistance.",
      "evidence": "Short excerpt or summary",
      "source": "https://source.com/article",
      "publicationDate": "2025-01-10",
      "confidence": "medium"
    }
  ],
  "coverageGaps": ["Limited data on independent artists"]
}
```

Why it matters:

- easier merge/compare
- preserves attribution/provenance
- gap detection is possible
- safer context passing to synthesis
- metadata is less likely to be dropped

## 14) Provenance in multi-agent systems

**Provenance** = tracking where each claim came from.

For each claim, you want metadata like:

- source URL / document name
- page number (for PDFs)
- publication date
- which subagent found it
- confidence/quality signal

Why it matters: if provenance is lost upstream, synthesis cannot reliably reconstruct it.

## 15) Error handling in multi-agent systems

Subagents can fail (timeout, access denied, missing inputs, etc.). Errors should be **structured** and include partial results when possible.

Bad:

- “Search failed.”

Good (example shape):

```json
{
  "isError": true,
  "errorCategory": "transient",
  "failureType": "timeout",
  "attemptedQuery": "AI in film production 2025",
  "partialResults": [
    { "claim": "AI is used in VFX workflows.", "source": "source_url" }
  ],
  "alternatives": [
    "retry with a narrower query",
    "use document analysis sources",
    "continue and mark a coverage gap"
  ]
}
```

Coordinator options:

- retry
- continue with partial results + caveat
- spawn another agent
- mark explicit coverage gaps

## 16) Full real-life example: research report

User request:

- “Create a report about how AI affects creative industries. Include music, film, publishing, and gaming. Use sources and mention uncertainty.”

Step 1: coordinator analyzes the request

- broad scope, multiple domains
- needs provenance/citations
- synthesis required
- parallelize domain research

Step 2: coordinator spawns parallel subagents (example prompts)

- Task(web_search_agent, “Research AI in music. Return claims, evidence, URLs, dates.”)
- Task(web_search_agent, “Research AI in film. Return claims, evidence, URLs, dates.”)
- Task(web_search_agent, “Research AI in gaming. Return claims, evidence, URLs, dates.”)
- Task(document_agent, “Analyze provided publishing reports. Return claims, evidence, document names, page numbers.”)

Step 3: subagents return structured findings (example)

Music:

```json
{
  "topic": "music",
  "findings": [
    {
      "claim": "AI tools assist with composition and production.",
      "source": "music-source-url",
      "date": "2025-01-10"
    }
  ],
  "coverageGaps": ["limited data on independent artists"]
}
```

Film:

```json
{
  "topic": "film",
  "findings": [
    {
      "claim": "AI is used in VFX and pre-production workflows.",
      "source": "film-source-url",
      "date": "2024-11-03"
    }
  ]
}
```

Publishing:

```json
{
  "topic": "publishing",
  "findings": [
    {
      "claim": "Publishers use AI for editing and summarization.",
      "document": "publishing-report.pdf",
      "page": 12
    }
  ]
}
```

Gaming timeout with partials:

```json
{
  "isError": true,
  "errorCategory": "transient",
  "failureType": "timeout",
  "attemptedQuery": "AI in gaming industry 2025",
  "partialResults": [
    { "claim": "AI is used for NPC behavior and asset generation.", "source": "gaming-source-url" }
  ],
  "alternatives": ["retry with narrower query", "continue with partial coverage"]
}
```

Step 4: coordinator handles the error

- retry with narrower scope (e.g., NPC behavior + dev tooling only)

Step 5: coordinator sends findings to the synthesis agent (explicit context passing)

Bad:

- “Summarize all findings.”

Good:

- “You are the synthesis agent. Use ONLY these findings: [paste the JSON blocks].”
- Requirements: preserve attribution; compare industries; mention uncertainty + gaps; do not invent claims; return structured report.

Step 6: synthesis returns a draft

- patterns across sectors
- sector differences
- risks/uncertainty
- explicit coverage gaps

Step 7: coordinator checks quality + decides whether to iterate

## 17) Full real-life example: codebase analysis

User: “Understand how authentication works in this codebase and suggest where to add 2FA.”

Coordinator decomposes:

- find auth entry points
- trace login flow
- inspect user/session model
- inspect frontend auth UI/state
- identify 2FA insertion points + risks

Subagents:

- Task(code_search_agent, “Find auth-related files with Grep/Glob.”)
- Task(backend_analysis_agent, “Trace login API and session creation.”)
- Task(frontend_analysis_agent, “Analyze login UI and auth state.”)
- Task(security_agent, “Identify 2FA integration risks.”)

Then coordinator merges results into the final recommendation.

## 18) Common exam questions

Type 1: synthesis ignores prior web findings

- Likely issue: **coordinator did not explicitly pass prior findings** to the synthesis agent.

Type 2: coordinator cannot spawn subagents

- Likely issue: `allowedTools` missing **`Task`**.

Type 3: report misses major areas, subagents “worked”

- Likely issue: **task decomposition too narrow**.

Type 4: high latency with independent subtasks

- Best answer: **run subagents in parallel** (multiple Task calls in one response).

Type 5: why hub-and-spoke?

- Best answer: observability, consistent error handling, controlled information flow.

Type 6: subagent timeout with partial results

- Best answer: return **structured error** with attempted input + partial results + alternatives.

## 19) Important traps

Trap 1: “Subagents share memory automatically.”

- Wrong: **Subagents do not automatically inherit parent context.**

Trap 2: “Always run all subagents.”

- Wrong: choose subagents based on task complexity; small tasks don’t need orchestration.

Trap 3: “Give every agent all tools.”

- Wrong: tool scoping reduces misuse and improves role alignment.

Trap 4: “Synthesis can recover lost sources.”

- Wrong: if provenance is dropped upstream, synthesis cannot reliably reconstruct it.

Trap 5: “If one subagent fails, stop everything.”

- Wrong: handle locally (retry, partial results, caveats, coverage gaps).

## 20) Key terms

| Term | Meaning |
| --- | --- |
| Multi-agent system | coordinator + specialized subagents |
| Coordinator | agent that delegates and aggregates |
| Subagent | specialized agent with narrow scope |
| Task tool | mechanism to spawn/invoke subagents |
| allowedTools | configuration that gates tool access |
| Explicit context passing | manually passing needed context into a subagent prompt |
| Parallel subagents | running independent subagent tasks concurrently |
| Iterative refinement | iterate until coverage/quality criteria are met |
| Hub-and-spoke | all subagent comms flow through coordinator |
| Task decomposition | splitting a complex task into subtasks |
| Coverage gaps | missing required scope areas |
| Provenance | source attribution for claims |

## 21) How to answer exam questions (fast checklist)

If the question says:

- subagent missed prior findings → **explicitly pass context**
- coordinator cannot invoke subagents → **add `Task` to allowedTools**
- final report misses whole domains → **decomposition too narrow**
- independent tasks are slow → **parallelize**
- subagents communicate chaotically → **hub-and-spoke via coordinator**
- synthesis lost sources → **require structured claim↔source mappings**

## 22) Repo note (for these notes)

This lecture lives in: `exam-notes/06-multi-agent-systems.md`.

## 23) Flashcards

Flashcards for this lecture are in: `flashcards/multi-agent-flashcards.md`.

## 24) Minimal summary (memorize)

- Coordinator = delegates, passes context, aggregates, checks coverage, decides to iterate.
- Subagent = specialized executor with scoped tools + structured outputs.
- Task tool = how the coordinator spawns subagents.
- Hub-and-spoke = all communication through coordinator.
- Explicit context passing = subagents only know what you paste into their prompt.
- Parallel subagents = lower latency for independent subtasks.
- Iterative refinement = check gaps → run more subagents → re-synthesize.

Most important sentence:

**Subagents do not automatically inherit parent context.**

