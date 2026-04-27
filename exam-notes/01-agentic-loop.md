# Agentic Loop

## 1. Short definition

**Chatbot** = user sends message, model returns text answer.

```text
User → Claude → Text answer

Agentic system = model can reason, call tools, receive tool results, and continue until task is finished.

User → Runtime/App → Claude → tool_use → Runtime executes tool → tool_result → Claude → final answer

Agentic loop is the repeated process where Claude decides whether to call a tool or finish the response.
```

## 2. Important mental model

Claude does not directly call your backend.

Claude returns a structured request like:

I want to call tool `lookup_order` with input `{ order_id: "123" }`.

Then the runtime executes the tool.

The runtime can be:

Environment	Who runs the loop
Claude website	Anthropic platform
Claude Code	Claude Code CLI
Cursor	Cursor agent/runtime
Your own app with Claude API	Your backend
Agent SDK app	Agent SDK runtime + your code

## 3. Basic flow

1. Send request to Claude
2. Claude returns response
3. Check stop_reason
4. If stop_reason = "tool_use" → execute tool
5. Add tool result to conversation history
6. Send updated conversation back to Claude
7. Repeat
8. If stop_reason = "end_turn" → finish

Detailed version:

User writes a message.
Runtime sends messages + tool definitions to Claude.
Claude either returns final text or asks to call a tool.
If Claude returns tool_use, runtime executes the tool.
Runtime adds both the assistant tool_use and tool_result to messages.
Runtime sends updated messages back to Claude.
Claude uses the tool result to decide the next step.
The loop continues until Claude returns stop_reason = "end_turn".

## 4. Key terms

Term	Meaning
Agent	A system around a model that can use tools and perform actions
Tool	A function/API/action exposed to Claude
tool_use	Claude asks the runtime to execute a tool
tool_result	The result of the executed tool
stop_reason	Field that explains why Claude stopped generating
end_turn	Claude finished and final answer can be returned
Runtime	The layer that sends requests, executes tools, stores messages
Messages array	Conversation history sent to Claude
Agentic loop	Claude → tool_use → tool_result → Claude → final answer

## 5. stop_reason

The two most important values:

stop_reason	What to do
tool_use	Execute the requested tool and continue the loop
end_turn	Stop the loop and return Claude's final answer

Correct stopping logic:

Primary stop condition: stop_reason == "end_turn"
Safety guard: max_iterations to prevent infinite loops

max_iterations is only an emergency brake. It should not be the main way to stop the loop.

## 6. Pseudocode

while true:
    response = send_to_claude(messages)

    if response.stop_reason == "tool_use":
        tool_call = response.tool_use
        result = execute_tool(tool_call)

        messages.append(tool_call)
        messages.append(tool_result)

        continue

    if response.stop_reason == "end_turn":
        return response.content

Meaning:

Send current conversation history to Claude.
Claude returns either tool request or final answer.
If Claude wants a tool, execute it.
Add the tool call and result to messages.
Send updated messages back to Claude.
Repeat until Claude returns end_turn.

## 7. Messages array

Claude API is usually stateless. It does not automatically remember everything between API calls.

So the runtime must send the conversation history every time.

Example before tool call:

[
  {
    "role": "user",
    "content": "Refund order #123"
  }
]

Claude returns tool_use:

{
  "type": "tool_use",
  "id": "toolu_1",
  "name": "lookup_order",
  "input": {
    "order_id": "123"
  }
}

Runtime executes the tool and gets result:

{
  "status": "delivered",
  "refund_eligible": true
}

Runtime updates messages:

[
  {
    "role": "user",
    "content": "Refund order #123"
  },
  {
    "role": "assistant",
    "content": [
      {
        "type": "tool_use",
        "id": "toolu_1",
        "name": "lookup_order",
        "input": {
          "order_id": "123"
        }
      }
    ]
  },
  {
    "role": "user",
    "content": [
      {
        "type": "tool_result",
        "tool_use_id": "toolu_1",
        "content": "{\"status\":\"delivered\",\"refund_eligible\":true}"
      }
    ]
  }
]

If the runtime does not add the tool result to messages, Claude will not know what the tool returned.

Possible consequences:

Claude calls the same tool again
Claude answers without data
Claude hallucinates
Claude makes a wrong decision

## 8. Tools

A tool can be:

internal backend function
API endpoint wrapper
database query
external API
built-in Claude Code tool

Examples:

get_customer
lookup_order
process_refund
escalate_to_human
search_web
Read
Write
Edit
Bash
Grep
Glob

Claude only sees the tool name, description, and input schema.

The runtime decides how the tool is actually executed.

Example:

Claude asks for lookup_order.
Runtime executes GET /api/orders/:id.
Runtime returns the result to Claude as tool_result.

## 9. Claude Code vs Claude API

Claude Code

In Claude Code, the loop is already implemented.

You → Claude Code CLI → Claude API → Claude Code executes Read/Grep/Edit/Bash → Claude API → You

You do not manually add tool_result.

Claude Code does it internally.

Your own Claude API app

In your own app, your backend/runtime manages the loop.

User → Frontend → Backend → Claude API → Backend executes tools → Claude API → Backend → Frontend

In this case, you must manage:

messages
tool definitions
tool execution
tool_result
stop_reason

## 10. Anti-patterns

Bad: stop by natural language
If Claude says "I am done", stop the loop.

This is bad because natural language is unreliable.

Claude can say:

I am done.
Task completed.
No further action is needed.
Here is the result.

Correct:

Stop when stop_reason == "end_turn".

Bad: stop only after fixed number of iterations
Stop after 5 iterations.

This is bad because some tasks need 2 tool calls and some need 7.

Correct:

Use stop_reason = "end_turn" as the primary condition.
Use max_iterations only as a safety guard.

Bad: not appending tool_result

If the tool result is not added to messages, Claude does not know what happened.

Correct:

Add assistant tool_use and tool_result to messages before the next Claude call.

## 11. Decision tree vs agentic loop

Decision tree

A decision tree is hardcoded logic.

if user asks about refund:
    call get_customer
    call lookup_order
    call process_refund

The developer defines the sequence.

Agentic loop

Claude dynamically decides the next tool based on context and tool results.

Claude reads the request.
Claude chooses a tool.
Runtime executes tool.
Claude observes result.
Claude chooses next step.

Decision Tree	Agentic Loop
Hardcoded steps	Model-driven steps
Less flexible	More adaptive
Good for predictable flows	Good for ambiguous tasks
Deterministic	Probabilistic unless guarded

Important exam idea:

Use model-driven reasoning for flexibility.
Use programmatic enforcement for critical business rules.

## 12. Programmatic enforcement

Prompt instruction:

Always verify customer before refund.

This can be placed in:

system prompt
Agent SDK agent definition
Claude Code CLAUDE.md

But prompt instructions are probabilistic.

For critical actions, use a programmatic guard:

Block process_refund unless verified_customer_id exists.

Examples:

block refunds above $500
block refund before customer verification
escalate policy exceptions to human
normalize tool results before Claude sees them

## 13. Multi-agent loop

A multi-agent system has a coordinator agent and specialized subagents.

Example:

Coordinator Agent
├── Web Search Agent
├── Document Analysis Agent
├── Synthesis Agent
└── Report Writer Agent

Flow:

User asks for research.
Coordinator decomposes the task.
Coordinator invokes subagents using Task tool.
Subagents return structured findings.
Coordinator passes findings to synthesis agent.
Synthesis agent creates draft report.
Coordinator checks coverage gaps.
Coordinator may call more subagents.
Coordinator returns final answer with end_turn.

Important:

Subagents do not automatically inherit coordinator context.
Context must be explicitly passed in the prompt.

Example:

Claude → Task(web_search_agent: AI in music)
Claude → Task(web_search_agent: AI in film)
Claude → Task(document_agent: AI in publishing)

Tool results → findings from subagents

Claude → Task(synthesis_agent with all findings)
Tool result → draft report

Claude → checks coverage gaps
Claude → maybe calls more subagents
Claude → end_turn final report

## 14. Grep tool

Grep searches inside file contents.

Examples:

Find all usages of useAuth.
Find where processRefund is called.
Find error message "Invalid token".

Glob searches file paths.

Examples:

Find all test files: **/*.test.tsx
Find all React components: src/components/**/*.tsx

Tool	Use
Grep	Search inside files
Glob	Find files by path/name pattern

## 15. Exam cheat sheet

Situation	Best answer
Need to continue after Claude asks for tool	Check stop_reason = "tool_use"
Need to finish response	Check stop_reason = "end_turn"
Claude does not know tool output	Tool result was not appended to messages
Claude skips customer verification	Add programmatic prerequisite / hook
Similar tools are confused	Improve tool descriptions
Need flexible task handling	Use agentic loop
Need deterministic business rule	Use guard/hook, not only prompt
Subagent needs previous findings	Pass context explicitly
Claude Code reads files	Built-in tools handle tool_result internally

