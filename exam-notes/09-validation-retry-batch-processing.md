# 09 — Validation, retry, batch processing (production pipelines)

This topic is beyond “prompt engineering”. It’s how real **extraction / review pipelines** work in production:

- PDF → extraction → validation → retry → save to DB
- code review → findings → validation → PR comments

## Extraction pipeline: mental model

1. Source document / diff
2. Claude extraction (ideally `tool_use` + schema)
3. **Syntax/schema validation**
4. **Semantic/business validation**
5. Retry if needed (with feedback)
6. Persist result / publish output

## Syntax validation (structure / schema)

Syntax validation checks whether the output is structurally usable:

- JSON is valid (no trailing commas, no extra text)
- required fields exist
- field types are correct
- enum values are valid
- overall schema compliance

Example: schema expects a number:

```json
{ "total": { "type": "number" } }
```

Model returns:

```json
{ "total": "one hundred" }
```

→ syntax/schema validation fails.

## Semantic validation (meaning / business correctness)

Semantic validation checks whether the values make sense:

- totals reconcile (line items + tax − discount = total)
- dates are real / plausible
- currency is supported
- values are not contradictory
- business rules are satisfied (limits, policy thresholds, permissions)

Examples:

- `subtotal + tax != total`
- `"invoiceDate": "2026-99-99"` (string is valid; date is impossible)
- `"currency": "APPLE"` (string is valid; value is nonsense)
- refund exceeds policy limit

## Key difference (exam)

| Validation type | Checks |
|---|---|
| **Syntax validation** | structure/format/schema correctness |
| **Semantic validation** | meaning/business correctness |

## Validation–retry loop (core production pattern)

Pipeline:

1. extract
2. validate
3. if invalid → retry with feedback
4. validate again
5. accept or fail permanently

Why retries help:

- missed fields
- ambiguous OCR
- arithmetic mistakes
- partial extraction

### Retry example (semantic mismatch)

Extraction 1:

```json
{ "subtotal": 900, "tax": 100, "total": 500 }
```

Validation fails:

```text
subtotal + tax must equal total
900 + 100 != 500
```

Retry feedback (good):

```text
Extraction failed validation:
- subtotal = 900
- tax = 100
- total = 500

These values do not add up.
Please re-check:
- whether subtotal was extracted correctly
- whether total includes tax
- whether a line item was missed
```

Extraction 2:

```json
{ "subtotal": 400, "tax": 100, "total": 500 }
```

Now valid.

## When retry helps vs does not help

Retry helps when the error is **recoverable**:

- missed line item
- misread digit
- tax/discount confusion
- formatting issues

Retry does **not** help when the source truly lacks information:

- “invoice has no tax ID”

Exam fix: use **nullable** fields (required + nullable is common) instead of forcing hallucination.

## Retry limits (avoid infinite loops)

Production systems always cap retries, e.g. `max_retries = 2`.

Pseudo-flow:

```text
for attempt in 1..(max_retries + 1):
  result = extract()
  if validate(result): return result
  feedback = build_validation_feedback(result)
fail permanently (or escalate)
```

## Batch processing (Message Batches API) vs synchronous

Batch APIs are for **large asynchronous workloads** where latency is acceptable.

### Latency-tolerant workloads

Meaning: the job does **not** need an immediate response (minutes/hours/overnight is fine).

Examples:

- overnight technical debt report
- analyze 5,000 support tickets
- extract data from 10,000 PDFs
- weekly code quality report

### When batch is NOT appropriate (exam)

If you need an immediate decision/response:

- chatbot reply
- interactive coding assistant
- **blocking pre-merge CI check**

Correct answer: use **synchronous** request/response.

## `custom_id` (batch correlation key)

In batch processing you must map results back to inputs. Use a stable `custom_id`.

Request item:

```json
{ "custom_id": "invoice_001", "message": "Extract invoice data" }
```

Response item:

```json
{ "custom_id": "invoice_001", "result": { "invoiceNumber": "INV-001" } }
```

## Exam traps (fast answers)

1. Need guaranteed correctness → JSON Schema alone is not enough → add **validation + retry loop**.  
2. JSON valid but totals inconsistent → **semantic** validation issue.  
3. Missing required field / wrong types / invalid JSON → **syntax/schema** validation issue.  
4. Process 100k docs overnight → **batch** API.  
5. Need immediate CI merge decision → **synchronous** API.  
6. Need to match batch responses to sources → **custom_id**.

## One-line exam answer

Production reliability = **extract → validate (syntax + semantic) → retry with specific feedback (bounded) → persist**, and choose **batch vs sync** based on latency requirements.

