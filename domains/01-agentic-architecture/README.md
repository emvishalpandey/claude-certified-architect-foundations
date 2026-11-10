# Domain 1 — Agentic Architecture & Orchestration (27%)

## 1.1 Agentic loop lifecycle

An agentic loop is the runtime cycle that lets Claude decide whether it needs a tool, receive the tool result, and continue reasoning. The application—not the model—executes the requested tool and sends the result back.

```text
messages = initial_messages
while True:
    response = call_claude(messages, tools=tool_definitions)
    messages.append(response)

    if response.stop_reason == "end_turn":
        return extract_final_answer(response)

    if response.stop_reason == "tool_use":
        for tool_call in response.tool_calls:
            result = execute_tool(tool_call.name, tool_call.input)
            messages.append(tool_result(
                tool_use_id=tool_call.id,
                content=result
            ))
        continue

    handle_other_stop_reason(response.stop_reason)
```

| Signal | Meaning | Runtime action |
| --- | --- | --- |
| stop_reason = tool_use | Claude has requested one or more tools | Execute the requested tool(s), append corresponding tool results, call Claude again |
| stop_reason = end_turn | Claude has completed its turn | Return the final response; stop the loop |
| Other stop reason | A different termination condition occurred | Handle explicitly according to API/runtime semantics; do not assume completion |

- Tool results must be linked to the corresponding tool-use request and appended to the conversation history.

- The next model call must include the conversation history needed for coherent reasoning.

- The model decides which tool to call based on context; this is different from a hard-coded sequence or decision tree.

- Do not parse phrases such as 'I'm done' to decide whether to terminate.

- Do not use assistant text content as the completion signal.

- An arbitrary iteration cap is not the primary termination mechanism. A defensive safety limit may exist, but it must not replace correct stop-reason handling.

| Exam pattern If the scenario says the agent stops after requesting a tool, inspect the loop. The likely defect is failure to execute the tool, append its result, and continue while stop_reason is tool_use. |
| --- |

## 1.2 Coordinator–subagent architecture

Use a hub-and-spoke design for multi-agent workflows. The coordinator decomposes the request, decides which specialists are needed, invokes them, handles errors, routes information and synthesizes the final result. Subagents do not automatically inherit the coordinator's conversation history.

| Component | Owns | Should not do |
| --- | --- | --- |
| Coordinator | Scope, task decomposition, delegation, context handoff, recovery, aggregation, final quality check | Delegate blindly to every agent on every request |
| Research/search subagent | Find sources and return claims plus source metadata | Write the final report without synthesis requirements |
| Document-analysis subagent | Extract evidence from supplied documents | Assume it can see documents or history not passed to it |
| Synthesis subagent | Combine supplied findings, preserve attribution, identify gaps | Invent missing evidence or silently reconcile conflicts |

Good coordinator behavior:

- Dynamically select subagents based on query complexity and required coverage.

- Decompose broad topics into a complete scope map before delegation. Avoid assigning only a narrow subset of a broad topic.

- Assign distinct subtopics or source types to reduce duplicated work.

- Pass the complete relevant findings and metadata explicitly in each subagent prompt.

- Launch independent subagents in parallel when their tasks do not depend on one another.

- Evaluate synthesis for missing coverage; issue targeted follow-up tasks where gaps remain.

- Route inter-agent communication through the coordinator for observability and consistent error handling.

```text
Coordinator task handoff (illustrative structure):
{
  "objective": "Assess the market and produce a cited report",
  "scope": ["North America", "Europe", "Asia-Pacific"],
  "agent_assignment": "Research the Europe segment only",
  "constraints": ["Use sources published within the last 24 months"],
  "output_contract": {
    "claims": [{"claim": "...", "evidence": "...",
                "source_url": "...", "published_at": "..."}],
    "coverage_gaps": []
  }
}
```

| Critical detail A subagent has isolated context. Put the necessary facts, prior findings, constraints and source metadata directly into its task prompt. Do not rely on implicit parent memory. |
| --- |

## 1.3 Spawning, tools and context passing

- The Task tool is the mechanism identified in the exam guide for spawning subagents.

- The coordinator must have Task included in allowedTools to invoke subagents.

- AgentDefinition describes each specialist: description, system prompt and tool restrictions.

- Multiple independent Task calls can be emitted in one coordinator response to run in parallel.

- Use structured outputs to separate content from metadata such as URLs, filenames, page numbers, timestamps and methodology.

- Fork-based sessions let you explore divergent approaches from a shared baseline without conflating their histories.

Bad handoff: 'Continue the research from the parent conversation.'

Good handoff: include the research question, scope, all relevant prior findings, evidence references, constraints, and the required output schema.

## 1.4 Workflow enforcement and human handoff

Prompt-based guidance is probabilistic. A prerequisite that protects money, identity, access or safety needs programmatic enforcement.

| Requirement | Recommended control |
| --- | --- |
| Verify identity before looking up private orders | Gate lookup_order until get_customer returns a verified customer ID |
| Block refund above policy threshold | Intercept the tool call in code/hook and deny or route to human review |
| Require approval before irreversible action | Explicit approval gate outside model discretion |
| Escalate a policy exception | Structured human handoff with case facts and recommended next action |

A useful handoff packet contains customer/case ID, verified identity status, issue summary, relevant transaction identifiers, amounts and dates, root cause, actions already attempted, tool results, policy gap, recommended action and urgency. A human should not need to reconstruct the case from a long transcript.

```text
Example policy gate:
if tool.name == "process_refund":
    require_verified_customer_id()
    if refund_amount > configured_limit:
        block_tool_call()
        create_human_escalation(reason="Refund exceeds authority")
    else:
        allow_tool_call()
```

## 1.5 Agent SDK hooks

| Hook pattern | Use | Why |
| --- | --- | --- |
| Pre-tool / tool-call interception | Inspect or block an outgoing tool call before execution | Enforce authorization, prerequisites, thresholds and policy |
| PostToolUse | Transform a tool result before the model consumes it | Normalize timestamps, status codes, inconsistent field formats |
| Other lifecycle hooks | Observe or influence defined runtime lifecycle points | Use only where the SDK exposes the relevant lifecycle event |

Use hooks when the requirement is deterministic. Example: normalize Unix timestamps to ISO 8601, map numeric status codes to canonical labels, or block a refund above $500. Prompt instructions can explain policy but cannot provide a hard guarantee.

| Exam trap If the scenario says 'must never', 'guarantee', 'block', 'before any financial operation', or 'policy threshold', choose programmatic enforcement rather than a stronger prompt. |
| --- |

## 1.6 Task decomposition

| Pattern | Use when | Example |
| --- | --- | --- |
| Prompt chaining / fixed sequential pipeline | Steps are predictable and order is known | Analyze each source file, then perform a cross-file integration review |
| Dynamic adaptive decomposition | The next task depends on discoveries made along the way | Explore a legacy system, map dependencies, then prioritize test gaps |
| Parallel decomposition | Tasks are independent and can be performed simultaneously | Research separate geographic markets or analyze separate documents |
| Coordinator refinement loop | Initial findings may be incomplete | Synthesize, detect missing coverage, delegate targeted research, synthesize again |

For large code reviews, use per-file local analysis followed by a separate cross-file pass. This limits attention dilution while retaining a view of integration and data-flow problems.

## 1.7 Session state, resumption and forking

| Approach | Use when | Risk / control |
| --- | --- | --- |
| Resume named session (--resume <session-name>) | Prior conversation and tool findings remain valid | Tell Claude which files changed since the session |
| Fork session | Explore competing designs from the same baseline | Keep branches independent and compare outputs |
| New session + structured summary | Prior tool results are stale or context is degraded | Inject current facts and findings rather than trusting old observations |

A resumed session can contain stale facts. If files changed, explicitly identify the changed paths and request targeted re-analysis. For long-running work, persist findings in scratchpad/state files and load a manifest on recovery.

---

*Source: supplied CCAR-F comprehensive study guide.*
