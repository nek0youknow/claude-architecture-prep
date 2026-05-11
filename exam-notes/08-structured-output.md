# 08 — Structured output (tool_use + JSON Schema)

## Core idea

**Structured output** means the model returns data in a **predictable machine-readable shape** (usually JSON), so downstream systems can reliably:

- store in a database
- send to backend APIs
- validate
- render in UI
- drive automation (CI, PR comments, workflows)
- extract fields from documents

Example (bad for parsing):

> The invoice number is INV-123, the total is 500 dollars, and the vendor is ABC Company.

Example (good for parsing):

```json
{
  "invoiceNumber": "INV-123",
  "totalAmount": 500,
  "currency": "USD",
  "vendor": "ABC Company"
}
```

## Why “return JSON” in a prompt is not enough

Even if you say “Return valid JSON”, the model can still produce:

- **invalid JSON** (syntax):
  - trailing commas
  - comments
  - unescaped characters
- **JSON wrapped in text** (“Here is the JSON: …”)
- **missing required fields**

So for reliability, prefer:

**tool_use + JSON Schema + validation + retry feedback**

## tool_use

`tool_use` means Claude returns a **structured tool call** with an `input` payload matching your schema, instead of free-form text.

You define a tool, e.g. `extract_invoice`, with an input schema like:

```json
{
  "type": "object",
  "properties": {
    "invoiceNumber": { "type": "string" },
    "totalAmount": { "type": "number" },
    "currency": { "type": "string" }
  },
  "required": ["invoiceNumber", "totalAmount", "currency"]
}
```

Then Claude returns:

```json
{
  "type": "tool_use",
  "name": "extract_invoice",
  "input": {
    "invoiceNumber": "INV-123",
    "totalAmount": 500,
    "currency": "USD"
  }
}
```

Downstream code reads the structured data from:

- `tool_use.input`

## Why tool_use + schema is the most reliable option

| Approach | Reliability |
|---|---|
| “Return JSON” in prompt | fragile (may break format) |
| Markdown JSON block | may include extra text around JSON |
| **tool_use + JSON Schema** | best for consistent shape |

Important: JSON Schema reduces **syntax/shape errors**, but does not guarantee **semantic correctness**.

## JSON Schema essentials

JSON Schema defines:

- fields (`properties`)
- types (`string`, `number`, `boolean`, arrays/objects)
- required vs optional (`required`)
- nullable fields (`type: ["string","null"]`)
- controlled vocab (`enum`)
- nested shapes (objects, arrays)

### Required vs optional

- **Required**: must be present in output shape.
- **Optional**: may be omitted entirely.

Use **required** when downstream systems always expect the key (e.g. `documentType`, `extractionStatus`).

Use **optional** when the field may truly be absent in the source (e.g. `middleName`, `discountCode`).

### Nullable (anti-hallucination)

Nullable means the key is present, but the value can be `null`.

Example value:

```json
{ "taxId": null }
```

Schema:

```json
{
  "taxId": { "type": ["string", "null"] }
}
```

Why this matters (exam trap): if you require a non-null string for data that may not exist, the model may fabricate.

Best pattern:

- **required + nullable**: key always exists; value is `null` if missing in source.

```json
{
  "type": "object",
  "properties": {
    "taxId": { "type": ["string", "null"] }
  },
  "required": ["taxId"]
}
```

## Enum (controlled vocabulary)

Enum restricts a field to a fixed set of allowed values.

```json
{
  "severity": {
    "type": "string",
    "enum": ["low", "medium", "high", "critical"]
  }
}
```

Why it helps: you avoid 100 variants like “pretty serious”, “very high”, “sev-1”, etc.

## `other` + detail (and `unclear`)

When an enum cannot cover everything, add:

- a value like `"other"` plus a detail field
- optionally `"unclear"` for ambiguous cases

Example:

```json
{
  "documentType": "other",
  "documentTypeOtherDetail": "medical bill"
}
```

Schema sketch:

```json
{
  "type": "object",
  "properties": {
    "documentType": {
      "type": "string",
      "enum": ["invoice", "receipt", "contract", "bank_statement", "other", "unclear"]
    },
    "documentTypeOtherDetail": { "type": ["string", "null"] }
  },
  "required": ["documentType", "documentTypeOtherDetail"]
}
```

Meaning:

- **other**: you know it’s not in the list, but you can name what it is
- **unclear**: the source is too ambiguous to classify safely

## `tool_choice`

`tool_choice` controls whether / which tool must be called:

- **auto**: model may answer in text or call a tool
- **any**: model must call *some* tool (choose among tools)
- **forced tool**: model must call a specific named tool

Exam mapping:

| Situation | tool_choice |
|---|---|
| normal chat | auto |
| structured output required but doc type unknown | any |
| specific extractor must run | forced tool |
| first step must be metadata extraction | forced extract_metadata |

## End-to-end example: invoice extraction

Tool: `extract_invoice`

Input schema pattern:

- most fields **required + nullable**
- enums for controlled vocab (`currency`, `documentType`, `severity`)

```json
{
  "type": "object",
  "properties": {
    "invoiceNumber": { "type": ["string", "null"] },
    "vendorName": { "type": ["string", "null"] },
    "invoiceDate": {
      "type": ["string", "null"],
      "description": "YYYY-MM-DD if present."
    },
    "totalAmount": { "type": ["number", "null"] },
    "currency": { "type": "string", "enum": ["USD", "EUR", "KZT", "GBP", "other", "unclear"] },
    "currencyOtherDetail": { "type": ["string", "null"] }
  },
  "required": [
    "invoiceNumber",
    "vendorName",
    "invoiceDate",
    "totalAmount",
    "currency",
    "currencyOtherDetail"
  ]
}
```

Model returns tool call:

```json
{
  "type": "tool_use",
  "name": "extract_invoice",
  "input": {
    "invoiceNumber": "INV-2026-001",
    "vendorName": "ABC Supplies",
    "invoiceDate": "2026-05-01",
    "totalAmount": 250000,
    "currency": "KZT",
    "currencyOtherDetail": null
  }
}
```

## Syntax errors vs semantic errors (exam-critical)

### Syntax / shape

Example: trailing comma → invalid JSON.

`tool_use + schema` mostly prevents this class of failures.

### Semantic correctness

JSON can be valid but wrong, e.g.:

- totals don’t match
- impossible date string
- contradictory fields

So you still need **business validation** after extraction.

## Validation + retry with feedback

After extraction, validate semantics:

- totals reconcile
- date is plausible
- currency supported
- no contradictions

If validation fails, retry with explicit feedback (only if the source likely contains the info):

- state what failed
- list the conflicting values
- suggest what to re-check (tax/discount/missed line item)

## CI/CD example: structured findings for PR comments

Use the same patterns for review output (findings array, severity enum, category enum with other+detail) so automation can post consistent PR comments.

## Common exam traps

1. Need guaranteed schema-compliant output → **tool_use + JSON Schema** (not “return JSON”).  
2. Field may be missing but schema requires string → make it **nullable** (or optional).  
3. Need controlled categories but allow unknown → **enum + other+detail (+ unclear)**.  
4. JSON valid but totals mismatch → **semantic validation + retry feedback**.  
5. Multiple extraction tools, doc type unknown, structured output required → `tool_choice: any`.  
6. Specific tool must run first → forced `tool_choice`.

## One-line exam answer

Need reliable structured output?

**Use tool_use with JSON Schema, then validate semantics and retry with feedback if needed.**

