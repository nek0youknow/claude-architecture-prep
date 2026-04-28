# Tools and MCP Flashcards

## Q: What is a tool?

A: A function/API/action Claude can ask the runtime to execute.

## Q: Does Claude execute the tool directly?

A: No. Claude returns tool_use. The runtime executes the tool.

## Q: What is an MCP tool?

A: An action exposed by an MCP server that Claude can call.

## Q: What is an MCP resource?

A: Context/catalog exposed by an MCP server, such as docs, schemas, or issue lists.

## Q: Tool vs Resource?

A: Tool performs an action. Resource provides context.

## Q: What is a tool description?

A: The description Claude uses to decide when and how to call a tool.

## Q: Why do models choose wrong tools?

A: Minimal descriptions, overlapping tool boundaries, bad names, too many tools, or biased system prompts.

## Q: What is tool boundary?

A: A clear explanation of what the tool is for and what it is not for.

## Q: What are similar tools?

A: Tools with overlapping names, inputs, or functionality.

## Q: If get_customer and lookup_order are confused, what is the best first fix?

A: Improve tool descriptions with boundaries, inputs, outputs, examples, and edge cases.

## Q: What is isError?

A: A structured flag indicating that a tool call failed.

## Q: Why not return generic "Operation failed"?

A: Claude cannot know whether to retry, ask user, escalate, or explain a business rule.

## Q: What is a transient error?

A: Temporary failure like timeout or service unavailable.

## Q: What is a validation error?

A: Invalid or missing input.

## Q: What is a permission error?

A: Access denied or insufficient permissions.

## Q: What is a business error?

A: Operation blocked by policy or business rule.

## Q: Empty results vs error?

A: Empty results can be a successful query with no matches. Error means the tool failed.

## Q: What is tool_choice auto?

A: Claude may call a tool or return text.

## Q: What is tool_choice any?

A: Claude must call a tool.

## Q: What is forced tool_choice?

A: Claude must call one specific named tool.

