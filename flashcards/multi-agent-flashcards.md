# Multi-agent Systems Flashcards

## Q: What is a multi-agent system?

A: A system with a coordinator agent and specialized subagents that complete parts of a task and return results for aggregation.

## Q: What does the coordinator do?

A: Decomposes the task, selects subagents, explicitly passes context, handles errors, aggregates results, checks coverage, and produces the final answer.

## Q: What is a subagent?

A: A specialized agent responsible for a narrow subtask with a role-scoped toolset and a defined output format.

## Q: What is hub-and-spoke architecture?

A: An architecture where all subagent communication is routed through the coordinator.

## Q: Why route all subagent communication through the coordinator?

A: Observability, consistent error handling, controlled information flow, and easier debugging.

## Q: What tool is used to spawn/invoke subagents?

A: The Task tool (conceptually).

## Q: What must the coordinator's allowedTools include to invoke subagents?

A: Task.

## Q: Do subagents automatically inherit coordinator/parent context?

A: No.

## Q: What is the key sentence to memorize about context?

A: Subagents do not automatically inherit parent context.

## Q: What is explicit context passing?

A: Manually including the necessary goal, scope, findings, metadata, constraints, and output format in the subagent prompt.

## Q: What should a coordinator pass to a synthesis subagent?

A: The findings themselves (ideally structured), plus provenance metadata, constraints (no invention), and the required output format.

## Q: When should subagents run in parallel?

A: When subtasks are independent and do not depend on each other’s outputs.

## Q: How do you reduce latency for independent subtasks?

A: Spawn parallel subagents by emitting multiple Task calls in a single coordinator response.

## Q: What is iterative refinement?

A: The coordinator checks synthesis output for coverage/quality gaps, runs additional targeted subagents, then re-synthesizes.

## Q: A report misses major domains but all subagents succeeded. What is the likely root cause?

A: The coordinator’s task decomposition was too narrow.

## Q: Coordinator invokes synthesis, but the synthesis ignores prior web findings. What is likely wrong?

A: The coordinator did not explicitly pass the prior findings into the synthesis prompt.

## Q: Why restrict tools per subagent?

A: To reduce misuse risk, improve tool selection, improve safety/auditability, and enforce role boundaries.

## Q: Why should subagents return structured output instead of messy text?

A: It makes merging easier, preserves provenance, enables gap detection, and supports reliable context passing.

## Q: What is provenance?

A: The traceable origin of each claim (source URL or document name/page, publication date, who found it, confidence).

## Q: What should a subagent return on failure (e.g., timeout)?

A: A structured error with failure type, attempted input, partial results (if any), and recommended alternatives.

## Q: What are partial results and why include them?

A: Useful findings returned before a failure; they allow the coordinator to continue, retry only missing parts, and mark coverage gaps.

