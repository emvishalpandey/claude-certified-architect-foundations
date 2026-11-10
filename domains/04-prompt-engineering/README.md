# Domain 4 — Prompt Engineering & Structured Output (20%)

## 4.1 Explicit criteria and precision

Vague instructions such as 'be conservative', 'review carefully' or 'only report high-confidence issues' do not define a decision boundary. Explicit criteria do.

| Weak instruction | Stronger instruction |
| --- | --- |
| Check comments for accuracy | Report a comment only when its claimed behavior contradicts the implementation; skip stylistic wording differences |
| Find serious bugs | Report defects that can cause incorrect persisted state, data loss, authorization bypass or externally visible incorrect behavior |
| Be conservative | Do not report formatting, naming preferences or patterns consistent with established local conventions |
| Use high severity appropriately | Critical = exploitable security issue or irreversible data loss; high = major functional failure with broad impact; medium = limited functional defect |

False positives reduce trust in the entire review system. If a category is persistently noisy, temporarily disable it while improving its criteria rather than flooding developers with low-value findings.

## 4.2 Few-shot prompting

Few-shot examples demonstrate the desired judgment and output format. They are especially useful for ambiguous cases, inconsistent document layouts, and distinguishing acceptable patterns from actual defects.

- Use 2–4 targeted examples for ambiguous decisions.

- Show why an action was chosen over plausible alternatives.

- Include both positive and negative examples: what to flag and what to skip.

- Demonstrate the exact output fields, severity labels and evidence format.

- Vary document structures so extraction generalizes beyond one template.

```text
Example review instruction:
Report only defects that meet one of these criteria:
1. Incorrect business behavior compared with the stated requirement.
2. Authorization or data-isolation failure.
3. Resource leak or unbounded resource consumption.
4. Breaking API contract change without compatibility handling.

Do not report formatting, naming preferences, or a pattern already
used consistently in the surrounding module.

For every finding return: file, line, category, severity,
evidence, impact, suggested fix.
```

## 4.3 Structured output with tool_use and JSON Schema

For reliable schema-constrained output, define a tool with JSON Schema parameters and use tool_use. The guide treats this as more reliable than asking Claude to print JSON in ordinary text.

| tool_choice | Guarantee |
| --- | --- |
| auto | Claude may respond with text rather than call a tool |
| any | Claude must call a tool, but can choose among available tools |
| Forced named tool | Claude must call the specific named tool |

Strict schemas eliminate JSON syntax errors, not semantic errors. A payload can conform to the schema while containing a wrong amount, an invalid date relationship or line items that do not sum to the total.

Schema design principles:

- Make truly mandatory fields required.

- Use optional or nullable fields when source information may be absent; this reduces pressure to fabricate.

- Use enums for stable categories. Add 'other' plus a detail field when categories may evolve.

- Add 'unclear' where the source is ambiguous.

- Specify normalization rules in the prompt (currency, dates, units, whitespace, identifiers).

- Validate relationships and business invariants in code after schema validation.

```text
Illustrative extraction schema:
{
  "type": "object",
  "properties": {
    "invoice_number": {"type": ["string", "null"]},
    "invoice_date": {"type": ["string", "null"]},
    "currency": {"type": "string", "enum": ["USD", "EUR", "GBP", "other"]},
    "currency_other_detail": {"type": ["string", "null"]},
    "stated_total": {"type": ["number", "null"]},
    "line_items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "description": {"type": "string"},
          "amount": {"type": "number"}
        },
        "required": ["description", "amount"]
      }
    }
  },
  "required": ["invoice_number", "invoice_date", "currency",
              "currency_other_detail", "stated_total", "line_items"]
}
```

Note: JSON Schema dialects and strict-mode requirements can differ by API version. The example illustrates the design pattern; check the current API schema constraints before copying into production.

## 4.4 Validation, retry and feedback loops

Use a validation pipeline with distinct stages:

- Schema validation: required fields, types, enum membership and structural constraints.

- Semantic validation: totals reconcile, dates are plausible, identifiers match expected formats, and extracted values are assigned to the correct fields.

- Source validation: every extracted fact is supported by the source and not inferred without authorization.

- Business validation: downstream rules and thresholds are satisfied.

On a resolvable failure, retry with the original document, failed extraction and exact validation errors. Retry can correct formatting or structural mistakes. It cannot recover information absent from the supplied source; retrieve the missing source or return null/unclear.

```text
Validation retry loop:
1. Extract structured fields.
2. Validate schema and semantic invariants.
3. If fixable:
   - send original source
   - send prior extraction
   - send precise validation errors
   - request corrected extraction
4. If source evidence is absent:
   - do not retry repeatedly
   - return null / unclear or route to human review
5. Record error category and correction outcome.
```

For automated code review feedback, include detected_pattern so dismissed findings can be grouped and analyzed. For extraction, return stated_total and calculated_total plus conflict_detected to make inconsistencies explicit.

## 4.5 Message Batches API

| Characteristic in supplied guide | Implication |
| --- | --- |
| 50% cost savings | Useful for high-volume, non-blocking workloads |
| Up to 24-hour processing window | Do not use for workflows that require an immediate result |
| No guaranteed latency SLA | Cannot be relied on for a blocking pre-merge gate |
| No multi-turn tool calling within one request | Not appropriate where the model must call tools, receive results and continue in the same batch request |
| custom_id | Correlates each response with its submitted request |

| Workload | Recommended API pattern |
| --- | --- |
| Pre-merge security/code check | Synchronous API / immediate response |
| Nightly technical debt report | Message Batches API |
| Weekly document classification | Message Batches API |
| Customer waiting for refund decision | Synchronous agent workflow with policy gates |
| Overnight test generation | Message Batches API if no blocking SLA |

Before a large batch, tune prompts on a representative sample. For failures, resubmit only failed custom_id items with appropriate changes, such as chunking oversized documents. Calculate submission cadence against the full SLA, including the maximum processing window and downstream review time.

## 4.6 Independent and multi-pass review

- A model that generated code retains its generation context; self-review may be biased toward its prior choices.

- Use a second independent Claude instance without the generator's reasoning context for a stronger review.

- For large changes, perform focused per-file reviews for local defects and a separate integration pass for cross-file data flow.

- Confidence scores can help route findings, but confidence must be calibrated and should not replace evidence or deterministic validation.

- Do not require multiple independent runs to agree on every bug; consensus filters can suppress intermittent but real defects.

---

*Source: supplied CCAR-F comprehensive study guide.*
