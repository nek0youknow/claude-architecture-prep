# Agentic Loop Flashcards

## Q: What is an agentic system?

A: A system where Claude can use tools, receive results, and continue reasoning until the task is complete.

## Q: What is a chatbot?

A: A system where the user sends a message and the model returns a text answer.

## Q: What does stop_reason = "tool_use" mean?

A: Claude wants the runtime to execute a tool.

## Q: What does stop_reason = "end_turn" mean?

A: Claude has finished and the final answer can be returned.

## Q: Does Claude directly call your backend?

A: No. Claude returns a tool_use request. The runtime/backend executes the actual tool.

## Q: What is the runtime?

A: The layer that sends messages to Claude, executes tools, appends tool results, and repeats the loop.

## Q: In Claude Code, who manages the agentic loop?

A: Claude Code CLI.

## Q: In your own app with Claude API, who manages the agentic loop?

A: Your backend or Agent SDK runtime.

## Q: What happens if tool_result is not added to messages?

A: Claude will not know the tool output and may repeat the tool, hallucinate, or make a wrong decision.

## Q: What is the primary stop condition?

A: stop_reason == "end_turn".

## Q: What is max_iterations for?

A: Safety guard to prevent infinite loops, not the main stopping condition.

## Q: What is a decision tree?

A: Hardcoded logic where the developer defines steps like if refund then call get_customer, lookup_order, process_refund.

## Q: Agentic loop vs decision tree?

A: Agentic loop is model-driven and adaptive; decision tree is hardcoded and deterministic.

## Q: Why is "Always verify customer before refund" not enough?

A: Prompt instructions are probabilistic. Critical rules need programmatic enforcement.

## Q: What is Grep?

A: A tool for searching inside file contents.

## Q: What is Glob?

A: A tool for finding files by path/name patterns.

## Q: What is multi-agent?

A: A system with a coordinator agent and specialized subagents.

## Q: Do subagents automatically inherit coordinator context?

A: No. Context must be explicitly passed.

