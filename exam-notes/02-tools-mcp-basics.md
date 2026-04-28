# Tools, Tool Descriptions, MCP Basics

## Core idea

Claude chooses tools based on:

- tool name
- tool description
- input schema
- system prompt
- conversation context
- previous tool results

If tools are chosen incorrectly, first check whether tool descriptions are clear.

## MCP Tool

An MCP tool is an action Claude can call through an MCP server.

Examples:

- get_customer
- lookup_order
- process_refund
- search_jira_issues
- query_database

## MCP Resource

An MCP resource exposes available context or catalogs.

Examples:

- documentation hierarchy
- database schema
- issue summaries
- policy documents

Tool = action.
Resource = context/catalog.

## Good tool description includes

- purpose
- when to use
- when not to use
- input formats
- output fields
- edge cases
- boundaries vs similar tools

## Tool boundary

A tool boundary explains what the tool does and does not do.

Example:

- get_customer is for customer profile and identity verification.
- lookup_order is for order status, delivery, payment, and refund eligibility.

## Similar tools

Similar tools need clear names and descriptions.

Bad:

- analyze_content
- analyze_document

Better:

- extract_web_results
- analyze_pdf_document
- summarize_policy_document

## isError

Tool failures should return structured errors.

Example:

```json
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "message": "Order service timed out."
}
```

## Error categories

| Category | Meaning | Retry |
|---|---|---|
| transient | temporary failure | yes |
| validation | bad input | no, fix input |
| permission | access denied | no |
| business | policy/business rule | no |

## Exam rule

If the model confuses get_customer and lookup_order because descriptions are minimal:

Correct fix:

- Improve tool descriptions.

Usually wrong first fixes:

- add many few-shot examples
- build routing classifier
- merge tools immediately

