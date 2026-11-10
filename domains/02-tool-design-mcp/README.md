# Domain 2 — Tool Design & MCP Integration (18%)

## 2.1 Design tool interfaces for reliable selection

Tool descriptions are a primary signal Claude uses to decide which tool to call. A name and one-line description are insufficient when tools overlap.

| Description element | What to specify |
| --- | --- |
| Purpose | Exact job the tool performs |
| Inputs | Accepted identifiers, formats, required and optional fields |
| Examples | Representative natural-language requests and valid input examples |
| Boundaries | What the tool does not do; when a similar tool is preferred |
| Output | Returned fields, units, status meanings and empty-result behavior |
| Side effects | Whether the operation reads, writes, charges, refunds, deletes or triggers external work |

Example of differentiated descriptions:

```text
get_customer:
"Resolve and verify a customer identity using an exact customer ID
or verified email. Returns customer_id and verification_status.
Use before accessing account-specific orders. Do not use this tool
to retrieve an order by order number."

lookup_order:
"Retrieve order status and line items using an exact order number.
Use for requests such as 'where is order 12345?' This tool does not
verify customer identity and must not be used to authorize refunds."
```

- If descriptions are ambiguous, improve them before adding a routing classifier.

- Split a generic tool into purpose-specific tools when their contracts or side effects differ.

- Rename overlapping tools to make their scope obvious.

- Review system prompt wording for keyword-sensitive instructions that accidentally bias tool selection.

- Do not combine unrelated operations merely to reduce tool count if it weakens boundaries or authorization.

## 2.2 MCP errors and recovery

MCP tool failures should be structured. A generic 'operation failed' gives the agent no basis for deciding whether to retry, ask for corrected input, explain a policy denial or escalate.

| Error category | Example | Retry? | Agent behavior |
| --- | --- | --- | --- |
| Transient | Timeout, service unavailable, temporary network failure | Usually yes, bounded retry/backoff | Retry or use an alternative; preserve partial results |
| Validation | Malformed order ID, missing required input | No, until input is corrected | Ask for corrected input or explain required format |
| Business | Refund exceeds policy, order is non-refundable | No | Explain policy or escalate if exception requested |
| Permission | Caller lacks required role or access | Not blindly | Do not retry unchanged; request authorization or escalate |
| Empty result | Valid query but no matching records | Not an error | Report no match; request another identifier if needed |

```text
Illustrative MCP error payload:
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "message": "Order service timed out",
  "attemptedOperation": "lookup_order",
  "partialResults": null,
  "suggestedAlternative": "Retry with the same order ID"
}
```

The guide specifically calls out the MCP isError flag and structured metadata such as errorCategory, isRetryable and a human-readable description. Keep the distinction between a failed query and a successful query that returned zero matches.

Subagents should recover locally from transient failures where appropriate. Escalate only unresolved errors, with what was attempted and any partial results.

## 2.3 Tool distribution and tool_choice

| Setting / pattern | Meaning | Use |
| --- | --- | --- |
| tool_choice: auto | Claude may choose a tool or respond with text | Normal flexible agent interaction |
| tool_choice: any | Claude must call a tool, but can choose which available tool | Guarantee structured tool use when the correct schema is one of several options |
| Forced tool selection | Require a named tool | Ensure a specific first operation, such as metadata extraction |
| Scoped tool access | Give each agent only role-relevant tools | Reduce selection complexity and prevent specialist misuse |

The guide warns that too many tools increase decision complexity. A specialist often needs four or five relevant tools, not every tool in the system. Give a synthesis agent a narrow verify_fact tool for simple checks if needed; route complex research through the coordinator.

| Least privilege Tool availability is an architectural boundary. Restricting tools reduces accidental misuse and limits the impact of a mistaken model decision. |
| --- |

## 2.4 MCP servers, tools and resources

MCP is the integration protocol used to expose tools and resources to Claude-enabled applications. In this exam, distinguish the two:

| MCP concept | Purpose | Example |
| --- | --- | --- |
| Tool | An operation the agent can invoke | Create ticket, retrieve order, run query, process refund |
| Resource | A content object/catalog the client can read | Issue summaries, documentation hierarchy, database schema catalogue |
| Server | A provider that exposes tools/resources | Jira integration server or internal operations server |

| Configuration scope | Location in exam guide | Typical purpose |
| --- | --- | --- |
| Project-level | .mcp.json | Shared team integration configuration committed with the repository |
| User-level | ~/.claude.json | Personal or experimental server configuration |

- Use environment-variable expansion such as ${GITHUB_TOKEN}; do not commit secrets.

- Configured servers' tools are discovered at connection time and can be available simultaneously.

- Expose catalogs and hierarchies as resources to reduce exploratory calls.

- Prefer an established community MCP server for standard integrations; build custom servers for team-specific workflows.

- Write descriptions that explain the MCP tool's capabilities and outputs so the agent does not default to a less capable built-in tool.

Illustrative project configuration (shape only; validate exact current syntax against the current Claude Code documentation):

```text
{
  "mcpServers": {
    "work-tracker": {
      "command": "npx",
      "args": ["-y", "example-mcp-server"],
      "env": {
        "API_TOKEN": "${WORK_TRACKER_TOKEN}"
      }
    }
  }
}
```

## 2.5 Built-in tools: choose the right one

| Tool | Best use | Example |
| --- | --- | --- |
| Glob | Find paths matching file patterns | Find all **/*.test.tsx files |
| Grep | Search file contents for text/patterns | Find every call to ProcessRefund or a specific error string |
| Read | Load file contents | Inspect a controller, service or test file |
| Edit | Targeted replacement anchored on unique text | Change one known method or configuration block |
| Write | Create or replace file contents | Create a new file or rewrite after a safe read |
| Bash | Execute shell commands | Run tests, inspect git diff, build or execute approved commands |

For codebase exploration, start with Glob/Grep to find likely entry points, then Read to trace imports, callers and dependencies. Do not read every file upfront. If Edit fails because the target text is not unique, Read the file and use a controlled Write or a more specific unique anchor.

---

*Source: supplied CCAR-F comprehensive study guide.*
