# Structured Output Flashcards

## Q: What is structured output?

A: Output returned in a predictable machine-readable format (often JSON) with a stable shape.

## Q: Why is “return JSON” in a prompt not enough?

A: The model may add text around JSON, omit fields, or produce invalid JSON (syntax errors).

## Q: What is the most reliable way to get guaranteed structured output?

A: **tool_use + JSON Schema** (then semantic validation).

## Q: What is tool_use?

A: Claude returns a structured tool call with an `input` object matching a schema (instead of free text).

## Q: Where is the structured data in tool_use?

A: Inside the tool call `input`.

## Q: What does JSON Schema define?

A: Fields, types, required/optional keys, nullable types, enums, and nested objects/arrays.

## Q: What is a required field?

A: A key that must be present in the output shape.

## Q: What is an optional field?

A: A key that may be omitted because it may truly be absent in the source.

## Q: What is nullable?

A: The key is present but the value can be `null` (e.g. type `["string","null"]`).

## Q: Why use nullable fields?

A: To avoid forcing the model to invent missing information.

## Q: What is “required + nullable”?

A: A best pattern where the key must exist, but can be `null` if missing in source (stable shape, less hallucination).

## Q: What is enum?

A: A list of allowed values for a field (controlled vocabulary).

## Q: Why use enum?

A: Prevents inconsistent labels (e.g. “pretty serious” vs “high”).

## Q: What is “other + detail”?

A: Use an enum value like `"other"` plus a detail field explaining what it is.

## Q: What is “unclear” used for?

A: When the source is ambiguous and cannot be safely classified.

## Q: What does tool_choice: auto mean?

A: Claude may call a tool or respond in text.

## Q: What does tool_choice: any mean?

A: Claude must call some tool (choose among available tools).

## Q: What is forced tool_choice?

A: Claude must call a specific named tool.

## Q: If multiple extraction tools exist and structured output is required, what tool_choice is useful?

A: `tool_choice: any`.

## Q: If a specific extraction must happen first, what should you use?

A: Forced tool_choice (force that tool).

## Q: Does JSON Schema prevent semantic errors?

A: No. It mostly prevents syntax/shape errors; semantic validation is still needed.

## Q: Example of a semantic error?

A: Line items total does not match the stated total (JSON valid, meaning wrong).

## Q: What should you do after semantic validation fails?

A: Retry with specific validation feedback if the information likely exists in the source; otherwise mark unknown/escalate.

