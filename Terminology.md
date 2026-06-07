# AI Agent Repository Terminology Guide

## Skills, plugins, workflows, subagents, metadata files, templates, prompts, resources, tools, hooks, and related concepts

This document gives a platform-neutral way to think about the building blocks used by modern AI coding agents such as Claude Code, Codex, IBM Bob, Cursor, Copilot-style agents, and internal enterprise agents.

Different tools use different names, but most agent systems are built from the same conceptual layers:

```text
Project context      → What the agent must know
Rules/config         → How the agent should behave
Tools/resources      → What the agent can access
Skills/workflows     → How the agent should perform repeated tasks
Subagents/modes      → Who should perform specialized tasks
Hooks/guardrails     → What must happen automatically or be blocked
Plugins/extensions   → How capabilities are packaged and shared
```

Official docs use similar patterns: Codex describes skills as packages of instructions, resources, and optional scripts; Claude Code describes plugins as a packaging layer for skills, hooks, subagents, and MCP servers; IBM Bob describes skills as reusable instruction sets and custom rules as project-specific behavior guidance. ([OpenAI Developers][1])

---

# 1. High-level taxonomy

| Term                              | What it means                       | Best used for                                                        | Do not use it for                    |
| --------------------------------- | ----------------------------------- | -------------------------------------------------------------------- | ------------------------------------ |
| Metadata file                     | Always-loaded project context       | Repo map, commands, standards, architecture notes                    | Long workflows, huge docs, secrets   |
| Rule / custom instruction         | Behavioral constraint               | Coding style, testing policy, security rules                         | Complex multi-step tasks             |
| Prompt                            | One-time instruction to the model   | Current task, question, command                                      | Permanent project knowledge          |
| Prompt template                   | Reusable prompt pattern             | Repeatable manual requests                                           | Enforced automation                  |
| Skill                             | Reusable task capability            | PR review, ADR writing, debugging, release checklist                 | Always-on project policy             |
| Workflow                          | Multi-step process                  | Feature implementation, release process, content generation pipeline | Single atomic tool call              |
| Subagent                          | Specialized worker agent            | Security review, test review, architecture analysis                  | Simple tasks or always-on rules      |
| Custom mode                       | Specialized behavior profile        | Read-only architect, docs-only writer, test engineer                 | Deep reusable workflow by itself     |
| Plugin / extension                | Packaged reusable capability bundle | Share skills, agents, hooks, MCP across repos/teams                  | One-off local instruction            |
| Tool / function                   | Executable action                   | Read file, run command, call API, query DB                           | Project knowledge storage            |
| MCP server / external tool server | Standardized external tool bridge   | GitHub, Figma, Sentry, docs, DB, APIs                                | Simple text instructions             |
| Resource                          | Data the agent can read/use         | Templates, docs, checklists, examples, schemas                       | Secrets unless controlled            |
| External resource                 | Data outside repo                   | API docs, tickets, logs, cloud data, design files                    | Untrusted truth without verification |
| Hook                              | Deterministic lifecycle automation  | Block commands, scan secrets, run checks                             | Reasoning-heavy subjective review    |
| Guardrail / permission            | Safety boundary                     | Prevent destructive commands, restrict files                         | Teaching the agent workflow          |
| Memory                            | Persistent remembered context       | User/team preferences, recurring facts                               | Large changing docs                  |
| Output schema                     | Required output shape               | JSON, reports, tickets, PR descriptions                              | Flexible brainstorming               |
| Evaluation                        | Quality check for agent output      | Regression testing agent behavior                                    | Runtime instruction                  |

---

# 2. Metadata files

## What they are

A **metadata file** is a project-level instruction file that gives the agent persistent context about the repository.

Examples:

```text
AGENTS.md       # Codex, IBM Bob, many interoperable tools
CLAUDE.md       # Claude Code
.cursor/rules   # Cursor-style rules
GEMINI.md       # Gemini-style project guidance
```

In Codex and IBM Bob, `AGENTS.md` is the important standard. Codex loads `AGENTS.md` as project guidance, and IBM Bob supports `AGENTS.md` as a team-standardized project instruction file. Claude Code uses `CLAUDE.md` for persistent project memory and project instructions. ([OpenAI Developers][1])

## What should go inside

Use metadata files for stable facts:

```text
- Project purpose
- Tech stack
- Repository map
- Build/test/lint commands
- Coding conventions
- Testing expectations
- Security constraints
- Git/PR behavior
- Important architecture notes
```

## Example

````md
# AGENTS.md

## Project purpose

This repository contains the frontend and backend services for [product].

## Tech stack

- Frontend: React, TypeScript
- Backend: Go
- Database: PostgreSQL
- Infrastructure: Kubernetes, Helm, Terraform

## Development commands

```bash
npm run lint
npm test
npm run build
````

## Engineering rules

* Prefer minimal diffs.
* Follow existing project patterns.
* Do not introduce new dependencies without explaining why.
* Do not commit or push unless explicitly asked.
* Never read or print secrets.

````

## When to use

Use a metadata file when the information should be available in almost every agent session.

## When not to use

Do not use metadata files for:

```text
- Long runbooks
- Full API documentation
- Large architecture documents
- Multi-step release procedures
- Secrets
- Rarely used workflows
````

If the metadata file becomes too long, move detailed processes into skills, workflows, or supporting resources.

---

# 3. Rules and custom instructions

## What they are

Rules are persistent behavioral constraints. They tell the agent **how to behave**.

IBM Bob calls these custom rules and says they can define coding style, documentation formats, testing methodologies, project workflows, and team conventions. ([IBM Bob][2])

Examples:

```text
.bob/rules/
.claude/rules/
.codex/rules/
.cursor/rules/
```

## Good rule examples

```md
# Testing rules

- Add tests for new behavior.
- Prefer focused unit tests before broad integration tests.
- Reuse existing test utilities.
- If tests cannot run locally, explain why.
```

```md
# Security rules

- Never read `.env` or secret files.
- Never hardcode tokens.
- Treat auth, IAM, migration, and infrastructure changes as high-risk.
```

## When to use

Use rules when the guidance is:

```text
- Stable
- Project-wide or mode-specific
- Behavioral
- Short enough to be loaded often
```

## When not to use

Do not use rules for complex procedures. For example, a full release process should be a skill or workflow, not a rule.

---

# 4. Prompts

## What they are

A **prompt** is the immediate instruction given to an agent.

Example:

```text
Review this branch against main and identify correctness, security, and test coverage risks.
```

Prompts are task-specific and short-lived.

## Types of prompts

| Prompt type      | Purpose                                    |
| ---------------- | ------------------------------------------ |
| User prompt      | What the user asks now                     |
| System prompt    | Platform-level behavior                    |
| Developer prompt | Tool/team-level behavior                   |
| Project prompt   | Repo guidance from metadata files          |
| Tool prompt      | Instruction attached to a tool or function |
| Subagent prompt  | Specialized worker instruction             |
| Skill prompt     | Reusable task instruction                  |

## When to use prompts

Use prompts for the current task, especially when the task is unique.

## When not to use prompts

If you repeat the same prompt often, convert it into:

```text
- a skill
- a slash command
- a workflow
- a template
```

---

# 5. Prompt templates

## What they are

A prompt template is a reusable prompt pattern with placeholders.

Example:

```md
Review the following change:

Context:
{{context}}

Changed files:
{{changed_files}}

Review focus:
{{review_focus}}

Return:
1. Summary
2. Critical issues
3. Test gaps
4. Suggested fixes
```

## When to use

Use prompt templates when humans still trigger the task, but the wording should be consistent.

Good cases:

```text
- PR review prompt
- Bug investigation prompt
- ADR writing prompt
- Blog generation prompt
- Release note prompt
```

## When not to use

Do not use prompt templates when the process requires file access, scripts, checklists, or tool use. Use a skill or workflow instead.

---

# 6. Skills

## What they are

A **skill** is a reusable capability package. It usually contains:

```text
- skill metadata
- instructions
- examples
- checklists
- templates
- reference files
- optional scripts
```

Codex describes skills as packages of instructions, resources, and optional scripts. IBM Bob describes skills as reusable instruction sets or recipes for specialized workflows. Claude Code also supports skills for reusable capabilities. ([OpenAI Developers][1])

## Typical structure

```text
skills/
└── review-pr/
    ├── SKILL.md
    ├── checklist.md
    ├── severity-guide.md
    └── examples/
        └── good-review.md
```

## Example `SKILL.md`

```md
---
name: review-pr
description: Review a branch or pull request for correctness, security, maintainability, and missing tests.
---

# Review PR

## Steps

1. Inspect changed files.
2. Understand the intended behavior.
3. Review correctness first.
4. Review security and maintainability.
5. Identify missing tests.
6. Suggest minimal fixes before refactors.

## Output

1. Summary
2. Critical issues
3. High-risk issues
4. Medium-risk issues
5. Test gaps
6. Recommended next actions
```

## When to use a skill

Use a skill when the task is:

```text
- Repeated often
- Procedural
- More than a simple prompt
- Useful across sessions
- Useful across team members
- Needs supporting files
```

Examples:

```text
review-pr
debug-failing-test
implement-feature
write-adr
audit-terraform-change
prepare-release-notes
generate-test-plan
```

## When not to use a skill

Do not use a skill for:

```text
- Always-on project context
- One-off tasks
- Hard safety enforcement
- Very small prompt shortcuts
```

Use metadata files for always-on context, hooks/permissions for enforcement, and prompt templates or slash commands for small shortcuts.

---

# 7. Workflows

## What they are

A **workflow** is a multi-step process. It may be implemented as:

```text
- a skill
- a slash command
- an automation scenario
- a graph of agents
- a CI pipeline
- a Make/Zapier/n8n workflow
- a custom program
```

A workflow is conceptual. A skill is one possible implementation of a workflow.

## Example workflow: feature implementation

```text
1. Understand the requirement
2. Inspect existing code
3. Propose implementation plan
4. Modify code
5. Add tests
6. Run focused test
7. Run lint/build
8. Summarize changes and risks
```

## When to use workflows

Use workflows for processes with ordered steps, decision points, or multiple tools.

Good cases:

```text
- Feature implementation
- Release readiness
- PR review
- Incident analysis
- Content research pipeline
- Migration planning
- Security review
```

## When not to use workflows

Do not over-engineer simple tasks. If the task is “rename this variable” or “explain this function,” a workflow is unnecessary.

---

# 8. Subagents

## What they are

A **subagent** is a specialized agent used by a parent agent to perform part of a task. It usually has:

```text
- its own role
- its own instructions
- restricted tools
- sometimes its own context window
- sometimes isolated filesystem or branch/worktree access
```

Codex supports spawning specialized agents in parallel and collecting their results. Claude Code has subagents as specialized assistants with their own context and tool access. ([OpenAI Developers][3])

## Example subagents

```text
security-reviewer
test-engineer
repo-architect
devops-reviewer
docs-writer
performance-reviewer
dependency-reviewer
```

## Example subagent definition conceptually

```yaml
name: security-reviewer
description: Reviews code and infrastructure for security risks.
tools:
  - read
  - grep
  - shell-readonly
instructions: |
  Focus on secrets, auth, authorization, input validation, dependency risk,
  infrastructure risk, and data leakage.
  Do not edit files.
  Return severity, evidence, exploitability, and remediation.
```

## When to use subagents

Use subagents when:

```text
- The task benefits from specialization
- Work can happen in parallel
- You want context isolation
- You want role-specific tools or permissions
- You want multiple independent reviews
```

Good cases:

```text
- PR review split into security, tests, correctness, performance
- Large codebase exploration
- Migration planning
- Incident investigation
- Architecture design review
```

## When not to use subagents

Do not use subagents for:

```text
- Simple one-step tasks
- Tiny code edits
- Always-on instructions
- Tasks where coordination overhead exceeds value
```

Subagents can increase cost, latency, and complexity. Start with one agent and add subagents only when the benefit is clear.

---

# 9. Custom modes

## What they are

A **custom mode** is a specialized behavior profile for the main agent. IBM Bob uses custom modes heavily. Modes can define role, behavior, when to use, and tool access. IBM Bob’s built-in modes include Code, Ask, Plan, Advanced, and Orchestrator. ([IBM Bob][2])

A custom mode is similar to a subagent, but usually it changes the active agent’s behavior instead of spawning a separate worker.

## Example modes

```text
architect-reviewer
docs-writer
security-reviewer
test-engineer
read-only-explorer
```

## When to use custom modes

Use custom modes when you want a reusable working style:

```text
- read-only architect mode
- docs-only writer mode
- test-only implementation mode
- planning mode
- security review mode
```

## When not to use custom modes

Do not use custom modes for detailed multi-step workflows. Use a skill for that.

A good pattern is:

```text
Custom mode = who the agent is
Skill       = what process the agent follows
```

Example:

```text
Mode: Security Reviewer
Skill: audit-auth-change
```

---

# 10. Plugins and extensions

## What they are

A **plugin** is a packaged bundle of agent capabilities. Claude Code documentation describes plugins as the packaging layer that can bundle skills, hooks, subagents, and MCP servers. ([Claude Code][4])

A plugin may include:

```text
- skills
- subagents
- hooks
- MCP server config
- templates
- commands
- resources
- scripts
```

## Example plugin structure

```text
team-engineering-plugin/
├── plugin.json
├── skills/
│   └── review-pr/
│       └── SKILL.md
├── agents/
│   └── security-reviewer.md
├── hooks/
│   └── hooks.json
├── resources/
│   └── severity-guide.md
└── scripts/
    └── collect-diff.sh
```

## When to use plugins

Use plugins when you need distribution and reuse:

```text
- same skills across many repos
- team-wide review process
- company security policies
- shared MCP integrations
- shared documentation templates
- reusable internal engineering kit
```

## When not to use plugins

Do not create a plugin for:

```text
- one repo only
- one developer only
- unstable experiments
- a single prompt
- one small skill
```

Start repo-local. Promote to a plugin after the pattern proves reusable.

---

# 11. Tools and functions

## What they are

A **tool** is an executable capability the agent can call.

Examples:

```text
read_file
write_file
run_shell_command
search_code
call_github_api
query_database
create_ticket
send_email
generate_image
run_tests
```

Tools are actions. They are not instructions.

## When to use tools

Use tools when the agent needs to interact with the world:

```text
- inspect code
- edit files
- run tests
- query logs
- call APIs
- create PRs
- update tickets
```

## When not to use tools

Do not use tools when static context is enough. For example, do not call an external API for something already documented in the repo unless freshness matters.

## Tool design rules

Good tools should be:

```text
- narrow
- auditable
- permissioned
- predictable
- well-documented
- safe by default
```

Bad tool design:

```text
do_anything(command: string)
```

Better tool design:

```text
run_tests(test_path: string)
read_github_pr(pr_number: int)
create_draft_pr(title: string, body: string)
```

---

# 12. MCP servers and external tool bridges

## What they are

MCP, or Model Context Protocol, is a common way to expose external tools and resources to agents.

Typical MCP integrations:

```text
- GitHub
- GitLab
- Jira
- Slack
- Google Drive
- Figma
- Sentry
- Datadog
- PostgreSQL
- internal docs
- cloud APIs
```

## When to use MCP or external tool servers

Use MCP when:

```text
- the agent needs live data
- the agent needs to call external APIs
- the integration should be reusable across tools
- you want a standard interface
- you need controlled tool permissions
```

## When not to use MCP

Do not use MCP for:

```text
- static project instructions
- simple templates
- one-off local scripts
- sensitive production actions without approvals
```

MCP gives capability. It does not automatically give safety. You still need permissions, approval, logging, and least-privilege design.

---

# 13. Resources

## What they are

A **resource** is information the agent can read but does not execute.

Examples:

```text
- architecture docs
- API specs
- templates
- checklists
- examples
- coding standards
- test plans
- design screenshots
- schema files
- runbooks
```

## Internal resources

Internal resources live inside the repo or agent package:

```text
docs/architecture.md
skills/review-pr/checklist.md
templates/adr.md
schemas/openapi.yaml
```

## External resources

External resources live outside the repo:

```text
- Confluence pages
- GitHub issues
- Jira tickets
- Slack threads
- Figma files
- Sentry logs
- cloud dashboards
- web documentation
```

## When to use resources

Use resources when the agent needs reference material, examples, or policy.

## When not to use resources

Do not dump every resource into context. Use retrieval, links, or progressive loading. Too much context reduces accuracy and increases cost.

---

# 14. Templates

## What they are

Templates are reusable output structures.

Examples:

```text
ADR template
PR description template
release notes template
incident report template
test plan template
blog outline template
architecture review template
```

## Example ADR template

```md
# ADR: {{title}}

## Status

Proposed | Accepted | Superseded

## Context

{{context}}

## Decision

{{decision}}

## Options considered

{{options}}

## Consequences

{{consequences}}

## Revisit criteria

{{revisit_criteria}}
```

## When to use templates

Use templates when the output format should be consistent.

## When not to use templates

Do not use rigid templates for exploratory thinking, brainstorming, or early-stage research where flexibility matters.

---

# 15. Hooks

## What they are

Hooks are deterministic actions that run at lifecycle events.

Claude Code documents hooks as shell commands, HTTP endpoints, or LLM prompts that run at specific lifecycle points. Codex also supports lifecycle hooks. ([Claude Code][5])

Examples:

```text
SessionStart
UserPromptSubmit
PreToolUse
PostToolUse
SubagentStart
SubagentStop
Stop
```

## Common hook use cases

```text
- block dangerous commands
- scan prompts for secrets
- run formatter after edits
- log tool usage
- check final completion criteria
- enforce branch policy
- prevent production actions
```

## When to use hooks

Use hooks when something must happen deterministically.

Good cases:

```text
- never allow terraform destroy
- block reading .env
- warn when tests fail
- run formatting after code edit
- scan for leaked secrets
```

## When not to use hooks

Do not use hooks for subjective reasoning:

```text
- “Is this architecture good?”
- “Is this blog engaging?”
- “Should we refactor?”
```

Use subagents, skills, or review workflows for subjective judgment.

---

# 16. Guardrails, permissions, and sandboxing

## What they are

Guardrails define what the agent is allowed to do.

Examples:

```text
- read-only mode
- workspace-write mode
- block network access
- require approval before shell commands
- deny reading secrets
- restrict editing to docs
- block production APIs
```

## When to use

Use guardrails for safety boundaries.

Good examples:

```text
- docs-writer can only edit .md files
- security-reviewer is read-only
- infrastructure changes require approval
- no access to .env files
- no push to remote
```

## When not to use

Do not rely only on natural language instructions for critical safety.

Weak:

```text
Please do not delete files.
```

Stronger:

```text
Block delete commands through permissions/rules/hooks.
Run in read-only sandbox.
```

---

# 17. Memory

## What it is

Memory is persistent context remembered across sessions.

Types:

```text
- user preferences
- team preferences
- project facts
- previous decisions
- coding style preferences
- recurring constraints
```

## When to use

Use memory for stable, reusable information.

Examples:

```text
- “Prefer minimal diffs.”
- “Use Carbon Design System.”
- “Do not include @param tags.”
- “Always produce PR summaries with risks and tests.”
```

## When not to use

Do not use memory for:

```text
- secrets
- temporary tasks
- large docs
- facts likely to change often
- sensitive personal data unless explicitly requested
```

Project-specific memory should usually live in the repo metadata file, not only in a personal assistant memory.

---

# 18. Output schemas

## What they are

An output schema defines the shape of the response.

Examples:

```json
{
  "summary": "...",
  "risks": [],
  "tests": [],
  "next_actions": []
}
```

or:

```md
## Summary

## Findings

## Tests

## Risks

## Recommendation
```

## When to use

Use output schemas when downstream automation depends on the response.

Good cases:

```text
- automation workflows
- CI agent reports
- PR review comments
- JSON pipelines
- release check reports
```

## When not to use

Do not over-constrain open-ended research, brainstorming, or ambiguous design work too early.

---

# 19. Evaluations

## What they are

Evaluations check whether an agent’s behavior is good enough.

Examples:

```text
- Did the agent run tests?
- Did it avoid editing forbidden files?
- Did it produce valid JSON?
- Did it identify known security issue?
- Did it avoid hallucinated APIs?
```

## When to use

Use evaluations when building repeatable agent workflows, skills, or plugins.

## When not to use

Do not create heavy evaluation infrastructure for a one-off personal prompt.

---

# 20. Artifacts

## What they are

Artifacts are files the agent produces.

Examples:

```text
- code files
- Markdown docs
- diagrams
- spreadsheets
- slides
- JSON configs
- PR descriptions
- test reports
```

## When to use

Use artifacts when the output needs to be stored, reviewed, versioned, or reused.

## When not to use

Do not create files when a short answer is enough.

---

# 21. Commands and slash commands

## What they are

Commands are named shortcuts for prompts or workflows.

Examples:

```text
/review
/create-pr
/release-check
/explain-module
/write-adr
```

## When to use

Use commands when humans frequently trigger the same action.

## Skill vs command

| Use command when           | Use skill when                     |
| -------------------------- | ---------------------------------- |
| It is a simple shortcut    | It is a full reusable capability   |
| No supporting files needed | Needs checklists/templates/scripts |
| Mostly manual invocation   | Can be auto-selected by the agent  |
| Lightweight prompt         | Structured process                 |

---

# 22. Configuration files

## What they are

Configuration files control model behavior, permissions, tools, hooks, skills, plugins, or project settings.

Examples:

```text
.codex/config.toml
.claude/settings.json
.bob/custom_modes.yaml
.bob/mcp.json
.mcp.json
```

## When to use

Use config files for machine-readable settings:

```text
- model
- sandbox
- permissions
- hooks
- MCP servers
- mode definitions
- approval policy
```

## When not to use

Do not put long prose instructions into config files unless the tool explicitly expects it. Use Markdown instruction files for human-readable guidance.

---

# 23. Choosing one over another

## If you need project context

Choose:

```text
AGENTS.md / CLAUDE.md / project metadata file
```

Not:

```text
skill, plugin, hook
```

## If you need repeatable task behavior

Choose:

```text
skill
```

Not:

```text
metadata file
```

## If you need a quick manual shortcut

Choose:

```text
slash command or prompt template
```

Not:

```text
plugin
```

## If you need specialized review

Choose:

```text
subagent or custom mode
```

Not:

```text
one giant skill
```

## If you need deterministic safety

Choose:

```text
permission, rule, sandbox, hook
```

Not:

```text
natural-language instruction only
```

## If you need external system access

Choose:

```text
MCP server or tool integration
```

Not:

```text
metadata file
```

## If you need team-wide distribution

Choose:

```text
plugin or reusable internal agent package
```

Not:

```text
copy-pasted prompts
```

## If you need consistent output

Choose:

```text
template or output schema
```

Not:

```text
free-form prompt
```

---

# 24. Practical decision tree

```text
Is this always-relevant project knowledge?
→ Put it in AGENTS.md / CLAUDE.md.

Is this a behavioral rule?
→ Put it in rules/custom instructions.

Is this a repeated process?
→ Make it a skill.

Is this just a short reusable prompt?
→ Make it a slash command or prompt template.

Does the task need a specialist role?
→ Use a subagent or custom mode.

Does the task need external tools/data?
→ Add a tool or MCP server.

Does something need to be enforced automatically?
→ Use permissions, sandboxing, rules, or hooks.

Does it need to be shared across many repos?
→ Package it as a plugin.

Does it need a consistent output format?
→ Use a template or output schema.

Does it need quality measurement?
→ Add evaluations.
```

---

# 25. Recommended repository layout for a mature AI-agent-ready repo

```text
repo/
├── AGENTS.md
├── README.md
├── docs/
│   ├── architecture.md
│   ├── testing.md
│   ├── security.md
│   └── release-process.md
├── .agent/
│   ├── rules/
│   │   ├── engineering-standards.md
│   │   ├── testing.md
│   │   └── security.md
│   ├── skills/
│   │   ├── review-pr/
│   │   │   ├── SKILL.md
│   │   │   ├── checklist.md
│   │   │   └── severity-guide.md
│   │   ├── implement-feature/
│   │   │   └── SKILL.md
│   │   ├── debug-failing-test/
│   │   │   └── SKILL.md
│   │   └── write-adr/
│   │       ├── SKILL.md
│   │       └── adr-template.md
│   ├── agents/
│   │   ├── security-reviewer.md
│   │   ├── test-engineer.md
│   │   └── repo-architect.md
│   ├── commands/
│   │   ├── review-branch.md
│   │   └── release-check.md
│   ├── templates/
│   │   ├── pr-description.md
│   │   ├── adr.md
│   │   └── incident-report.md
│   ├── hooks/
│   │   ├── block-dangerous-command.sh
│   │   └── secret-scan.sh
│   └── resources/
│       ├── coding-style.md
│       └── review-checklist.md
└── .mcp.json
```

Then map that generic structure to the tool:

| Generic        | Claude Code                     | Codex                | IBM Bob                                  |
| -------------- | ------------------------------- | -------------------- | ---------------------------------------- |
| metadata       | `CLAUDE.md`                     | `AGENTS.md`          | `AGENTS.md`                              |
| rules          | `.claude/rules/` or `CLAUDE.md` | `.codex/rules/`      | `.bob/rules/`                            |
| skills         | `.claude/skills/`               | `.agents/skills/`    | `.bob/skills/`                           |
| agents         | `.claude/agents/`               | `.codex/agents/`     | custom modes + subtasks                  |
| config         | `.claude/settings.json`         | `.codex/config.toml` | `.bob/custom_modes.yaml`, settings       |
| hooks          | `.claude/settings.json` hooks   | `.codex/hooks.json`  | not publicly documented as general hooks |
| plugins        | Claude plugins                  | Codex plugins        | MCP or reusable `.bob/` package          |
| external tools | MCP                             | MCP                  | MCP                                      |

---

# 26. Common mistakes

## Mistake 1: Putting everything in the metadata file

Bad:

```text
AGENTS.md contains 1,000 lines of rules, release process, templates, API docs, and examples.
```

Better:

```text
AGENTS.md = high-level project context
skills/ = workflows
resources/ = reference docs
templates/ = output formats
```

## Mistake 2: Using skills for safety enforcement

Bad:

```text
Skill says: “Never run terraform destroy.”
```

Better:

```text
Rules/permissions/hooks block terraform destroy.
Skill explains how to review Terraform changes.
```

## Mistake 3: Creating subagents too early

Bad:

```text
20 subagents before the workflow is proven.
```

Better:

```text
Start with 3:
- reviewer
- test-engineer
- security-reviewer
```

## Mistake 4: Creating plugins too early

Bad:

```text
Build a plugin for one repo and one unstable workflow.
```

Better:

```text
Start repo-local.
Promote to plugin after reuse across several repos.
```

## Mistake 5: Giving tools broad permissions

Bad:

```text
run_shell(command: string) with no approval.
```

Better:

```text
Separate tools:
- run_tests
- run_lint
- read_logs
- create_draft_pr
```

## Mistake 6: Confusing resources with truth

External resources can be stale, incomplete, or wrong. The agent should cite, compare, and verify them.

---

# 27. Recommended default strategy

For a new AI-agent-ready repo, start with five layers:

```text
1. Metadata file
2. Rules
3. Skills
4. Templates/resources
5. Safe tool configuration
```

Only add subagents, hooks, plugins, and external MCP integrations after the workflow proves useful.

## Minimal starting setup

```text
AGENTS.md
.agent/rules/testing.md
.agent/rules/security.md
.agent/skills/review-pr/SKILL.md
.agent/skills/debug-failing-test/SKILL.md
.agent/templates/pr-description.md
```

## Mature setup

```text
AGENTS.md
rules/
skills/
templates/
resources/
subagents/
hooks/
mcp/
plugins/
evals/
```

---

# 28. Final mental model

The cleanest way to think about all these terms is:

```text
Metadata file  = What the agent should know
Rules          = How the agent should behave
Prompt         = What I want now
Template       = What the output should look like
Skill          = How to do a repeated task
Workflow       = The process from start to finish
Subagent       = Who should handle a specialized part
Custom mode    = The personality/tool-access profile
Tool           = What the agent can do
Resource       = What the agent can read
MCP            = How the agent reaches external systems
Hook           = What must happen automatically
Guardrail      = What the agent must not be allowed to do
Plugin         = How to package and distribute capabilities
Evaluation     = How to measure whether it worked
```

For most serious software repositories, the best default is:

```text
Use metadata for stable context.
Use rules for behavior.
Use skills for repeatable work.
Use templates for consistent outputs.
Use subagents for specialist review.
Use hooks and permissions for safety.
Use MCP for external tools.
Use plugins only after reuse is proven.
```

[1]: https://developers.openai.com/codex/skills?utm_source=chatgpt.com "Agent Skills – Codex"
[2]: https://bob.ibm.com/docs/ide/configuration/rules?utm_source=chatgpt.com "Custom rules | Docs"
[3]: https://developers.openai.com/codex/subagents?utm_source=chatgpt.com "Subagents – Codex"
[4]: https://code.claude.com/docs/en/features-overview?utm_source=chatgpt.com "Extend Claude Code - Claude Code Docs"
[5]: https://code.claude.com/docs/en/hooks?utm_source=chatgpt.com "Hooks reference - Claude Code Docs"
