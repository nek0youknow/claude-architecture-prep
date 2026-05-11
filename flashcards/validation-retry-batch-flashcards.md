# Validation, Retry, Batch Processing Flashcards

## Q: What does syntax validation check?

A: Structure/format/schema: valid JSON, required fields, correct types, enum values, schema compliance.

## Q: What does semantic validation check?

A: Meaning and business correctness (totals reconcile, dates plausible, rules satisfied).

## Q: Example of a semantic validation issue?

A: Line items do not sum to the stated total (JSON is valid but wrong).

## Q: Example of a syntax/schema validation issue?

A: Invalid JSON, missing required key, wrong type (e.g. `"total": "one hundred"` when schema requires number).

## Q: What is a validation–retry loop?

A: Extract → validate → retry with feedback if invalid → validate again → accept or fail.

## Q: Why is retry useful?

A: It can fix recoverable extraction mistakes (missed fields, OCR ambiguity, arithmetic confusion).

## Q: When does retry NOT help?

A: When the source document truly does not contain the information.

## Q: What should you use instead of retrying missing fields forever?

A: Nullable fields (often **required + nullable**) to avoid hallucination.

## Q: Why use retry limits?

A: To avoid infinite loops and uncontrolled cost/latency.

## Q: What makes good retry feedback?

A: Specific failed checks + conflicting values + what to re-check (tax/discount/missed line item).

## Q: What is batch processing (Message Batches API) used for?

A: Large asynchronous, latency-tolerant workloads (overnight/at scale).

## Q: What is a latency-tolerant workload?

A: A workload that does not require an immediate response (minutes/hours/overnight is fine).

## Q: Example of a latency-tolerant workload?

A: Overnight technical debt report or mass document extraction.

## Q: When should you NOT use batch APIs?

A: Interactive systems or blocking checks that need immediate responses.

## Q: What API type should be used for blocking pre-merge CI checks?

A: Synchronous request/response.

## Q: What is `custom_id` used for in batches?

A: Correlating each batch result back to the original input item.

## Q: Exam: JSON valid but business logic wrong — what is it?

A: Semantic validation failure.

## Q: Exam: malformed JSON or schema mismatch — what is it?

A: Syntax/schema validation failure.

