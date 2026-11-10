# Domain 3 — Claude Code Configuration & Workflows (20%)

## 3.1 CLAUDE.md hierarchy and organization

| Scope | Location in guide | Who it affects |
| --- | --- | --- |
| User | ~/.claude/CLAUDE.md | Only the current developer; not shared by version control |
| Project | Root CLAUDE.md or .claude/CLAUDE.md | Team/project instructions when committed |
| Directory | CLAUDE.md in a subdirectory | Instructions relevant to that directory tree |

Use project-level instructions for shared standards: architecture conventions, build/test commands, code review expectations, fixture usage and repository-specific practices. Use user-level instructions for personal preferences. Use directory-level instructions for local ownership and conventions.

- Use @import to keep instruction files modular and include standards relevant to a package.

- Use .claude/rules/ for focused topic-specific rule files such as testing.md, api-conventions.md and deployment.md.

- Use /memory to inspect which memory files are loaded when behavior differs across sessions.

- Avoid a huge monolithic instruction file that mixes every package's conventions.

## 3.2 Commands and skills

| Mechanism | Location | When to use |
| --- | --- | --- |
| Project slash command | .claude/commands/ | Reusable team workflow committed to source control |
| Personal slash command | ~/.claude/commands/ | Developer-specific workflow |
| Project skill | .claude/skills/<skill>/SKILL.md | On-demand, reusable task procedure for the project |
| Personal skill | ~/.claude/skills/ | Personal variation; use a distinct name to avoid changing team behavior |
| CLAUDE.md | Project/user/directory locations | Always-loaded or scope-loaded instructions and standards |

The guide identifies skill frontmatter fields:

| Frontmatter | Purpose |
| --- | --- |
| context: fork | Run the skill in an isolated subagent context; useful for verbose exploration or brainstorming |
| allowed-tools | Restrict tools available during skill execution |
| argument-hint | Tell the user which parameters to provide |

Choose CLAUDE.md for universal instructions that should be present consistently. Choose a skill for a specific workflow that is invoked when needed. Use context: fork when the task's output is lengthy or exploratory and should not pollute the main session.

## 3.3 Path-specific rules

Rules in .claude/rules/ can use YAML frontmatter with glob path patterns. They load conditionally when the agent works on matching files. This is more maintainable than directory-only instructions when the same convention applies across files scattered throughout the repository.

```text
---
paths:
  - "src/api/**/*"
---
API conventions:
- Use the established async request pattern.
- Return the standard error contract.

---
paths:
  - "**/*.test.tsx"
---
Test conventions:
- Prefer user-visible behavior assertions.
- Reuse shared fixtures where available.
```

Use path-scoped rules for cross-cutting file types (for example all tests, Terraform files or API handlers) regardless of directory. Use a subdirectory CLAUDE.md when conventions are genuinely tied to that directory.

## 3.4 Plan mode versus direct execution

| Plan mode | Direct execution |
| --- | --- |
| Large multi-file changes | Small, clear, well-scoped change |
| Architecture or design decisions | Known implementation path |
| Migration, refactor, multiple valid approaches | Single validation check or isolated bug fix |
| Explore dependencies before editing | Implement immediately when scope and impact are understood |
| Use Explore subagent for verbose discovery | Avoid unnecessary planning overhead |

For a .NET monolith-to-microservices restructuring, a framework migration across dozens of projects, or a Terraform architecture change, use plan mode first. Explore the repository, map dependencies, identify risks and propose a sequence. Then execute the agreed plan. For a one-line validation fix with a known stack trace, direct execution is appropriate.

## 3.5 Iterative refinement

- Provide 2–3 concrete input/output examples when prose descriptions are interpreted inconsistently.

- Write tests for expected behavior, edge cases and performance requirements before implementation; feed failures back into the next iteration.

- Use the interview pattern: ask Claude to identify unanswered design questions and risks before implementation.

- Address interacting defects together; fix independent defects sequentially to simplify diagnosis.

- For edge cases, provide exact input and expected output rather than saying 'handle nulls correctly'.

Example: instead of 'normalize customer names', provide inputs such as '  smith, jane ' and expected output 'Jane Smith', and explicitly define handling of null, empty, initials, accents and multi-part surnames.

## 3.6 Claude Code in CI/CD

The exam guide emphasizes non-interactive execution and machine-readable output.

```text
claude -p "Review this pull request for security and correctness"

# Structured output for pipeline parsing:
claude -p "Review the changed files"   --output-format json   --json-schema ./review-schema.json
```

- Use -p / --print in CI to avoid waiting for interactive input.

- Use --output-format json and --json-schema for machine-parseable findings.

- Put repository testing standards, fixtures and review criteria in CLAUDE.md.

- Use an independent review session rather than asking the same context that generated code to review itself.

- When re-running after new commits, pass previous findings and ask for only new or still-unaddressed issues.

- Provide existing tests so generated tests do not duplicate coverage already present.

Example CI review schema fields: file_path, line_start, line_end, category, severity, finding, evidence, suggested_fix, detected_pattern. The schema must be validated by the consumer; syntactically valid JSON does not guarantee a true finding.

---

*Source: supplied CCAR-F comprehensive study guide.*
