# Domain 5 — Context Management & Reliability (15%)

## 5.1 Preserve critical facts across long interactions

Long conversations accumulate tool output and can degrade. Summaries are useful but can lose exact dates, amounts, identifiers and customer commitments. Preserve critical facts separately from narrative summaries.

| Context layer | What belongs there |
| --- | --- |
| Case facts | Exact IDs, amounts, dates, statuses, customer commitments, verification state |
| Conversation summary | Goals, decisions, unresolved questions and interaction history |
| Tool evidence | Only fields needed for the next decision, with source metadata |
| Scratchpad / state | Exploration findings, dependency maps, current phase, completed tasks and next actions |
| Provenance | Claim-to-source mapping, publication date, page or URL, evidence excerpt |

- Trim verbose tool output before adding it to the conversation. Keep only relevant fields.

- Put key findings near the beginning of aggregated inputs and use explicit section headings.

- Preserve exact transactional facts in a persistent case-facts block included in each prompt.

- Pass structured facts, citations and relevance scores downstream rather than verbose reasoning.

- Include full conversation history in subsequent API requests when needed for conversational coherence.

| Lost in the middle The guide highlights that information near the beginning and end of long inputs is often processed more reliably than information buried in the middle. Put key findings first, structure the rest with headings, and avoid unnecessary tool output. |
| --- |

## 5.2 Escalation and ambiguity resolution

| Situation | Correct behavior |
| --- | --- |
| Customer explicitly asks for a human | Honor the request immediately; do not force another investigation first |
| Straightforward issue within policy | Offer/perform autonomous resolution even if customer is frustrated, unless they reiterate human preference |
| Policy exception or policy silence | Escalate; do not invent a policy interpretation |
| No meaningful progress possible | Escalate with attempted actions and evidence |
| Multiple customer/account matches | Ask for additional identifiers; do not guess based on name or heuristics |
| Complex but clearly within policy and tools | Complexity alone is not an automatic escalation trigger |

Sentiment and self-reported confidence are poor proxies for case complexity. Use explicit criteria and examples. Distinguish a frustrated customer with a simple resolvable issue from a policy exception requiring authority.

## 5.3 Error propagation in multi-agent systems

A subagent should report enough information for the coordinator to choose a recovery strategy.

```text
{
  "status": "partial_failure",
  "failure_type": "timeout",
  "attempted_query": "latest regulatory filing for issuer X",
  "attempts": 2,
  "partial_results": [{"claim": "...", "source_url": "..."}],
  "alternatives": ["try issuer website", "narrow date range"],
  "coverage_gaps": ["latest filing not verified"]
}
```

- Do not convert a timeout into an empty successful result.

- Do not terminate the entire workflow because one independent subagent failed.

- Recover locally from transient errors; propagate unresolved failures with partial results.

- Annotate the final synthesis with coverage gaps and unsupported areas.

- A valid empty result is different from a failed attempt to access the source.

## 5.4 Large codebase exploration and recovery

- Use subagents to investigate bounded questions such as 'trace refund flow dependencies' or 'find all test files'.

- Keep the main agent focused on architecture and coordination rather than dumping every file into its context.

- Persist findings in scratchpad files so later phases can reuse them.

- Summarize each phase before starting the next; inject that summary into new subagent prompts.

- For crash recovery, have agents export structured state to known locations and let the coordinator load a manifest.

- Use /compact when a long session accumulates verbose discovery output, but verify that critical facts survive compaction.

## 5.5 Human review and confidence calibration

A 97% aggregate accuracy figure can hide a weak document type or field. Measure accuracy by segment before automating high-confidence decisions.

- Track performance by document type, field, source quality and relevant operating segment.

- Use field-level confidence scores, then calibrate thresholds against a labeled validation set.

- Route low-confidence, ambiguous or contradictory cases to human review.

- Use stratified random sampling of high-confidence extractions to estimate residual error and detect new failure patterns.

- Do not remove human review solely because aggregate accuracy appears high.

| Review segment | Example metric |
| --- | --- |
| Document type | Invoice vs contract vs purchase order accuracy |
| Field | Invoice number vs tax ID vs total accuracy |
| Confidence band | Observed precision for 0.90–0.95 model confidence |
| Error pattern | Missing currency, date ambiguity, line-item arithmetic mismatch |

## 5.6 Provenance and uncertainty in synthesis

Summarization can destroy the link between a claim and its source. Preserve claim-source mappings through every agent handoff.

```text
{
  "claim": "Revenue increased by 8%",
  "source": {
    "url": "https://example.org/report",
    "document": "Annual report",
    "page": 42,
    "published_at": "2026-03-15",
    "excerpt": "Revenue increased by..."
  },
  "data_period": "FY2025",
  "methodology_note": "Constant currency"
}
```

- Require publication or collection dates; different time periods may explain apparently conflicting values.

- When credible sources disagree, preserve both values and their source context. Do not arbitrarily select one.

- Separate established findings from contested findings.

- Preserve methodology and original source characterization.

- Report coverage gaps explicitly.

- Use the appropriate output format: financial data in tables, news in prose, technical findings in structured lists.

---

*Source: supplied CCAR-F comprehensive study guide.*
