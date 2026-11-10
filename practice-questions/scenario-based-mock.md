# Scenario-based mock questions

> Original practice questions reproduced from the supplied study guide; they are not actual exam questions.

# Practice exam — 25 scenario-based questions

Choose the number of responses requested. These are original practice questions based on the supplied blueprint, not actual exam questions.

1. A refund agent sometimes calls process_refund before identity verification. The system prompt already says verification is mandatory. What is the strongest fix? (Select one)

- A. Add more examples to the prompt

- B. Add a programmatic gate that blocks refund until verified identity is present

- C. Increase the model temperature

- D. Add a sentiment classifier

2. Claude requests a tool and the runtime immediately returns a final answer without executing it. What is broken? (Select one)

- A. The system prompt

- B. The loop fails to handle stop_reason=tool_use

- C. The tool schema is too strict

- D. The session name

3. A broad research topic only returns findings about two of six relevant sectors. All agents completed their assigned work. Most likely cause? (Select one)

- A. The coordinator decomposed the topic too narrowly

- B. The synthesis agent needs a larger model

- C. The search tool needs retries

- D. The output format is too short

4. A synthesis subagent says it cannot use findings found earlier by the coordinator. What should change? (Select one)

- A. Increase max_tokens

- B. Explicitly pass findings and metadata in the subagent prompt

- C. Ask it to remember

- D. Run it twice

5. Two MCP tools are repeatedly confused because their descriptions are nearly identical. Best first step? (Select one)

- A. Build a routing classifier

- B. Add more tools

- C. Rewrite descriptions with inputs, examples and boundaries

- D. Force one tool for every request

6. A search times out after returning partial results. What should the subagent return? (Select one)

- A. Empty success

- B. Generic unavailable

- C. Structured failure type, attempted query, partial results and alternatives

- D. Terminate all agents

7. Which pattern is appropriate for a synthesis agent that needs occasional simple fact checks? (Select one)

- A. Give it every search tool

- B. Give it a scoped verify_fact tool and route complex research through coordinator

- C. Remove all tools

- D. Let it fabricate likely values

8. A team-wide /review command must be available after cloning the repo. Where should it live? (Select one)

- A. ~/.claude/commands/

- B. .claude/commands/ in the project

- C. Root CLAUDE.md

- D. .mcp.json

9. Test files are scattered throughout the repository, but all need the same rule. Best approach? (Select one)

- A. Root CLAUDE.md prose only

- B. One CLAUDE.md per directory

- C. .claude/rules/ with a test-file glob

- D. Personal command

10. A 40-project .NET migration has architectural choices and cross-project dependencies. Best workflow? (Select one)

- A. Direct execution

- B. Plan mode and codebase exploration first

- C. Ask Claude to edit every file at once

- D. Avoid reading dependencies

11. A CI job hangs because Claude waits for input. Which option is identified in the guide? (Select one)

- A. --batch

- B. CLAUDE_HEADLESS=true

- C. -p / --print

- D. --silent

12. Which workflow is the best fit for Message Batches API? (Select one)

- A. Blocking pre-merge check

- B. Live refund approval

- C. Overnight technical debt report

- D. Interactive customer chat

13. A schema-valid extraction has line items totaling 950 while stated_total is 1,000. What is required? (Select one)

- A. Nothing; schema passed

- B. Semantic validation

- C. A larger context window

- D. Disable strict schema

14. A required field may be absent from the source. How should it be modeled? (Select one)

- A. Required non-null string

- B. Nullable/optional field with explicit absent-value handling

- C. Invent a likely value

- D. Retry indefinitely

15. A validation retry cannot find a tax ID because it is not in the supplied document. Best response? (Select one)

- A. Retry 20 times

- B. Return null/unclear or request the missing source

- C. Guess from company name

- D. Remove source validation

16. A model generates code and then reviews it in the same session. What is the concern? (Select one)

- A. It has too little context

- B. It retains generation reasoning and may be biased toward its prior decisions

- C. It cannot read code

- D. JSON output is invalid

17. A long case contains exact refund amounts and order IDs. How should they survive summarization? (Select one)

- A. Narrative summary only

- B. Persistent structured case-facts block

- C. Remove old messages

- D. Rely on model memory

18. Customer explicitly asks for a human agent. What should the agent do? (Select one)

- A. Investigate first

- B. Honor the request immediately

- C. Ask for sentiment score

- D. Attempt refund

19. A valid customer query returns no matching order. What is this? (Select one)

- A. Transient error

- B. Permission error

- C. Successful empty result

- D. Validation error

20. Two credible sources report different values for the same metric, measured in different periods. What should synthesis do? (Select one)

- A. Pick the newest-looking number

- B. Average them

- C. Preserve both with source and date context

- D. Drop both

21. Which tool finds all files ending in .cs? (Select one)

- A. Grep

- B. Glob

- C. Edit

- D. Bash only

22. Which tool finds every occurrence of ProcessRefund in file contents? (Select one)

- A. Glob

- B. Grep

- C. Write

- D. Read only

23. Which tool_choice requires a tool call but allows the model to choose among available tools? (Select one)

- A. auto

- B. any

- C. end_turn

- D. tool_use

24. Which is the best way to measure residual error in high-confidence extractions? (Select one)

- A. Trust model confidence

- B. Stratified random sampling of high-confidence outputs

- C. Review only low-confidence outputs

- D. Use aggregate accuracy only

25. A convention must apply to every Terraform file, wherever located. What should be used? (Select one)

- A. A directory-only CLAUDE.md

- B. Path-scoped rule with terraform glob

- C. User-level command

- D. A tool resource

# Practice exam answer key and reasoning

## Answer key and reasoning

# Practice exam answer key and reasoning

1. Answer: B. A prompt is probabilistic. A financial prerequisite needs deterministic enforcement.

2. Answer: B. Execute the requested tool, append its result and continue the loop.

3. Answer: A. The coordinator's scope decomposition determines coverage.

4. Answer: B. Subagents do not automatically inherit parent context.

5. Answer: C. Fix the source of ambiguity before adding infrastructure.

6. Answer: C. Structured context lets the coordinator recover intelligently.

7. Answer: B. Scoped cross-role access supports common simple cases without over-provisioning.

8. Answer: B. Project-scoped commands are version-controlled.

9. Answer: C. Glob path-scoped rules cover matching files regardless of directory.

10. Answer: B. Complex multi-file architectural work benefits from exploration and planning.

11. Answer: C. The print flag runs non-interactively.

12. Answer: C. Batch processing is latency tolerant and has no guaranteed latency SLA.

13. Answer: B. Schema validity does not guarantee semantic consistency.

14. Answer: B. Nullable fields reduce pressure to fabricate missing facts.

15. Answer: B. Retries cannot recover absent information.

16. Answer: B. Use an independent review instance.

17. Answer: B. Preserve exact transactional facts separately.

18. Answer: B. The guide says explicit human requests should be honored immediately.

19. Answer: C. An empty result is not an access failure.

20. Answer: C. Preserve provenance and temporal context; do not silently reconcile.

21. Answer: B. Glob matches file paths.

22. Answer: B. Grep searches content.

23. Answer: B. any forces a tool call without forcing a particular named tool.

24. Answer: B. Sampling high-confidence outputs detects hidden errors and novel patterns.

25. Answer: B. Path-scoped glob rules apply across directories.

# Hands-on preparation labs

## Additional advanced questions

# 7. Advanced practice questions — additional scenario traps

26. An agent repeats lookup_order with the same identifier after receiving an empty successful result. Best correction? (Select one)

- A. Treat every empty result as transient and retry

- B. Preserve empty-success semantics, detect repeated/no-progress behavior as a safety condition, then ask for a different identifier or escalate

- C. Increase max_tokens

- D. Change temperature

27. A workflow must call extract_intent before routing. A prompt instruction is skipped in 15% of runs. Strongest first-turn control? (Select one)

- A. tool_choice auto

- B. tool_choice any

- C. Force the named extract_intent tool

- D. Add more system prompt text

28. A generic tool handles refunds, store credit, exchanges and partial refunds through a mode field. Output quality is inconsistent. Best design? (Select one)

- A. Longer description only

- B. Split into purpose-specific tools with distinct contracts

- C. Increase context window

- D. Add a second model to select the mode

29. Project MCP configuration must be shared, but each developer has a different token. Correct approach? (Select one)

- A. Commit each token

- B. Move all config to user scope

- C. Commit project config with environment-variable references

- D. Remove authentication

30. An analysis skill must be read-only. What constrains its tool access? (Select one)

- A. context: fork

- B. allowed-tools limited to read/search tools

- C. argument-hint

- D. User-level CLAUDE.md

31. Customer is upset about a delayed order but has not requested a human. The issue is within policy and remedy is available. What should the agent do? (Select one)

- A. Escalate based on sentiment

- B. Acknowledge frustration and offer the permitted resolution

- C. Ask customer to repeat the complaint

- D. Route to supervisor because case is emotional

32. Source document contains no renewal date. Safest extraction output? (Select one)

- A. Infer from contract start date

- B. Return null/unclear and preserve missing-evidence state

- C. Retry until date appears

- D. Use common renewal period

33. Accuracy is 97% on English digital invoices, but production includes scans and German invoices. First evaluation improvement? (Select one)

- A. Add more English digital samples

- B. Segment evaluation using representative formats and languages

- C. Remove low-confidence cases

- D. Treat model confidence as accuracy

34. Search subagent fails while document analysis succeeds. Coordinator response? (Select one)

- A. Terminate report

- B. Treat search as empty success

- C. Use successful findings, consider fallback/retry and annotate coverage gap

- D. Ask synthesis to invent citations

35. A 20-file review misses cross-file data flow defects. Best redesign? (Select one)

- A. One much longer prompt

- B. Per-file analysis plus a separate integration pass

- C. Three full reviews and majority vote

- D. Disable cross-file checks

36. CI review repeats old comments after every new commit. What should be added? (Select one)

- A. Include prior findings and ask for only new or unaddressed issues

- B. Increase output tokens

- C. Change temperature

- D. Delete test suite

37. Tool returns 40 fields, but next step needs status, amount and order date. What should happen? (Select one)

- A. Append entire response

- B. Trim before adding to context

- C. Ask model to ignore fields

- D. Summarize all fields

38. Two source values differ. What metadata is essential before calling them contradictory? (Select one)

- A. Agent confidence

- B. Publication/data collection date and methodology/population

- C. Search query length

- D. Number of subagents

39. A long code exploration crashes. What supports robust recovery? (Select one)

- A. Chat history only

- B. Structured state exports and a manifest loaded by coordinator

- C. Re-run all commands blindly

- D. Increase temperature

40. A noisy review category damages trust in all findings. Best immediate action? (Select one)

- A. Keep reporting but lower severity

- B. Temporarily disable category while refining criteria

- C. Ask model to be careful

- D. Add more categories

# Advanced practice answer key

26. Answer: B. An empty successful lookup is not an access failure. Repeated no-progress calls need a bounded recovery path.

27. Answer: C. Forced named selection guarantees that tool; any only guarantees some tool.

28. Answer: B. Different operations and side effects warrant purpose-specific contracts.

29. Answer: C. Share integration configuration, not secret values.

30. Answer: B. allowed-tools is the permission surface; fork isolates context.

31. Answer: B. Frustration alone is not an escalation trigger.

32. Answer: B. Missing source information must not be fabricated.

33. Answer: B. Aggregate accuracy masks segment weakness; evaluation must mirror production.

34. Answer: C. Graceful degradation preserves partial value and exposes uncertainty.

35. Answer: B. Focused local passes preserve depth; integration pass catches cross-file behavior.

36. Answer: A. Prior findings enable deduplication across review runs.

37. Answer: B. Filter at the tool/runtime boundary before context accumulation.

38. Answer: B. Temporal and methodological context may explain differences.

39. Answer: B. A manifest is a durable checkpoint independent of chat history.

40. Answer: B. Suppress the noisy category until it meets acceptable precision.

# 8. Recommended practice-resource workflow