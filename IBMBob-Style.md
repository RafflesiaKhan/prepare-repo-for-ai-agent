Below is a research-backed report on **IBM Bob project repository setup**, with a practical repo structure you can use for your own engineering projects.

Important correction first: based on IBM Bob’s public docs, the project instruction file is **`AGENTS.md`**, not `IBMBOB.md` or `BOB.md`. Bob also uses `.bob/` for project-level rules, modes, skills, commands, and MCP configuration.

---

# 1. What IBM Bob is, in repo terms

IBM Bob is an agentic IDE for software development across the SDLC: writing code, testing, upgrading, securing, documenting, automating tasks, and working with real codebases. IBM describes Bob as an AI SDLC partner that augments existing workflows and works confidently with real codebases. ([IBM Bob][1])

For project configuration, Bob mainly uses:

```text
your-repo/
├── AGENTS.md
├── .bobignore
├── .bob/
│   ├── rules/
│   │   ├── 01-project-standards.md
│   │   ├── 02-testing.md
│   │   └── 03-security.md
│   ├── rules-code/
│   │   └── typescript-react.md
│   ├── rules-plan/
│   │   └── planning.md
│   ├── rules-docs-writer/
│   │   └── documentation-style.md
│   ├── custom_modes.yaml
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
│   ├── commands/
│   │   ├── review-branch.md
│   │   ├── release-check.md
│   │   └── explain-module.md
│   └── mcp.json
└── docs/
    ├── architecture.md
    ├── testing.md
    └── release-process.md
```

The key idea is:

```text
AGENTS.md           = persistent project knowledge
.bob/rules/         = behavior rules
.bob/custom_modes.yaml = specialized Bob personas/tool access
.bob/skills/        = reusable workflows
.bob/commands/      = slash commands
.bob/mcp.json       = project-level MCP tools
.bobignore          = file access boundaries
```

---

# 2. Does Bob use `AGENTS.md`?

Yes. Bob supports `AGENTS.md` as a team-standardized project instruction file. IBM’s custom rules documentation says `AGENTS.md` can be placed in the workspace root, is automatically loaded by default, can be version-controlled, and can be disabled through `"bob-code.useAgentRules": false` in settings. It is loaded after mode-specific rules but before general workspace rules. ([IBM Bob][2])

IBM also has a Bob tutorial specifically titled **“Start a project with /init and AGENTS.md”**. The search result summary states that `/init` scans a project and generates a structured `AGENTS.md` file that Bob uses as persistent context in every conversation. ([IBM Bob][3])

So the practical answer is:

```text
Use AGENTS.md for IBM Bob.
Do not invent IBMBOB.md unless your team has a separate convention.
```

## Recommended `AGENTS.md`

````md
# AGENTS.md

## Project purpose

This repository contains [short description of the system].
The main user/business outcome is [outcome].

## Tech stack

- Frontend: [React / Angular / Vue / etc.]
- Backend: [Node.js / Java / Go / Python / etc.]
- Database: [PostgreSQL / DynamoDB / DB2 / etc.]
- Infrastructure: [Kubernetes / OpenShift / Terraform / Helm / Argo CD / etc.]
- Tests: [Jest / Pytest / Go test / JUnit / etc.]

## Repository map

- `src/`: application source code
- `src/components/`: UI components
- `src/services/`: service/API layer
- `src/hooks/`: React hooks or reusable logic
- `tests/`: test files and fixtures
- `infra/`: infrastructure-as-code
- `docs/`: architecture and operational documentation

## Local development

Before adding or changing commands, inspect:

- `package.json`
- `Makefile`
- `README.md`
- `docs/local-development.md`

Common commands:

```bash
npm install
npm run lint
npm test
npm run build
````

## Coding standards

* Prefer minimal, focused changes.
* Follow existing project patterns.
* Do not introduce new dependencies without explaining why.
* Do not change public APIs unless required.
* Avoid formatting-only diffs unless formatting is the task.
* Do not delete tests unless they are obsolete and the reason is documented.

## Testing expectations

Before saying work is complete:

1. Run the smallest relevant test first.
2. Run lint or typecheck when touching typed code.
3. Run broader tests for cross-cutting changes.
4. If tests cannot be run, explain exactly why.

## Security and secrets

* Never read, print, or modify `.env`, `.env.*`, credentials, private keys, tokens, or production secret files.
* Never hardcode secrets.
* Treat auth, IAM, infrastructure, migration, payment, and production changes as high-risk.
* Do not run destructive infrastructure commands unless explicitly requested.

## Git behavior

* Do not commit unless explicitly asked.
* Do not push unless explicitly asked.
* Summarize changed files, tests run, and residual risks.
* For PR descriptions, include: summary, tests, risks, rollback notes.

## Communication

* State assumptions.
* Explain trade-offs for architectural changes.
* Prefer concrete findings over generic advice.

````

Keep this file concise. Put long workflows in `.bob/skills/`, not `AGENTS.md`.

---

# 3. `.bob/rules`: Bob custom rules

Bob custom rules are project or global instructions that influence Bob’s responses, coding style, documentation style, testing methods, workflows, and team conventions. IBM documents two scopes: global rules and workspace rules. Global rules apply across projects, while workspace rules apply only to the current project. :contentReference[oaicite:3]{index=3}

Bob supports simple file-based rules:

```text
.bobrules
.bobrules-code
.bobrules-{modeSlug}
````

and directory-based rules:

```text
.bob/
├── rules/
│   └── coding-style.md
├── rules-code/
│   └── typescript.md
└── rules-plan/
    └── planning.md
```

IBM recommends the directory approach for better organization. The docs say `.bob/rules/` contains general rules for all modes and `.bob/rules-code/` contains Code-mode-specific rules. ([IBM Bob][2])

## Rule priority

IBM documents this priority:

1. Global rules: `~/.bob/rules/`
2. Workspace rules: `.bob/rules/`

Workspace rules can override global rules. Within each level, mode-specific rules load before general rules. ([IBM Bob][2])

Bob also reads rule files recursively, processes files alphabetically, skips empty files, supports symlinks up to depth 5, and excludes cache-like files such as `.DS_Store`, `*.bak`, `*.cache`, `*.log`, `*.tmp`, and `Thumbs.db`. ([IBM Bob][2])

## Recommended `.bob/rules/01-project-standards.md`

```md
# Project standards

- Prefer existing project patterns over introducing new architecture.
- Make the smallest change that solves the task.
- Do not introduce a dependency unless the benefit is clear.
- Explain any public API changes.
- Preserve backward compatibility unless the task explicitly requires a breaking change.
- Avoid unrelated formatting changes.
```

## Recommended `.bob/rules/02-testing.md`

```md
# Testing rules

- Add or update tests for new behavior.
- Prefer focused unit tests before broad integration tests.
- Reuse existing test helpers and fixtures.
- Run the smallest relevant test first.
- If a test cannot run locally, explain why and provide the command that should be run.
```

## Recommended `.bob/rules/03-security.md`

```md
# Security rules

- Do not read, print, or modify secret files.
- Do not hardcode credentials or tokens.
- Treat authentication, authorization, IAM, infrastructure, migration, and production changes as high-risk.
- For high-risk changes, provide a risk summary and rollback note.
- Do not run destructive commands without explicit user approval.
```

## Recommended `.bob/rules-code/typescript-react.md`

```md
# Code mode rules for TypeScript and React

- Prefer TypeScript-safe changes.
- Avoid `any` unless necessary and explained.
- Prefer existing hooks and service abstractions.
- Keep components small and readable.
- Add tests for changed behavior.
- Follow existing Carbon Design System usage where applicable.
```

---

# 4. `.bobignore`: controlling Bob’s file access

Bob supports `.bobignore`, with syntax identical to `.gitignore`. Its purpose is to protect sensitive information, prevent accidental changes to build artifacts or large assets, and define Bob’s operational scope. IBM states that Bob monitors `.bobignore` and reloads changes automatically, and the `.bobignore` file itself is implicitly ignored so Bob cannot change its own access rules. ([IBM Bob][4])

Example:

```gitignore
# Secrets
.env
.env.*
*.pem
*.key
*.p12
*.pfx
secrets/
config/secrets.json
credentials.json

# Dependencies and build outputs
node_modules/
dist/
build/
coverage/
target/
.venv/
venv/

# Logs and caches
*.log
.cache/
tmp/
.DS_Store

# Large generated assets
*.zip
*.tar
*.tgz
*.mp4
```

IBM documents that `.bobignore` is strictly enforced by tools such as `read_file`, `write_to_file`, `apply_diff`, and `list_code_definition_names`, but it also documents a current limitation: `insert_content` and `search_and_replace` may bypass `.bobignore` during final save operations. IBM also states `.bobignore` is not a full system-level sandbox. ([IBM Bob][4])

That means `.bobignore` is necessary, but not sufficient. For sensitive repos, combine it with:

```text
- no auto-approve for Execute
- no auto-approve for Write unless in a disposable branch
- restricted custom modes
- code review before commit
- Git branch protection
- local OS-level secret hygiene
```

---

# 5. Custom modes

Bob modes are specialized personas. IBM says modes tailor Bob’s behavior for specific tasks, with different capabilities and access levels. Built-in modes include Code, Ask, Plan, Advanced, and Orchestrator. ([IBM Bob][5])

Bob’s own docs recommend using different modes for specialization, safety controls, focused interactions, and workflow optimization. ([IBM Bob][5])

## Built-in modes

From IBM’s docs:

| Mode         | Best use                                                     |
| ------------ | ------------------------------------------------------------ |
| Code         | Write, modify, refactor code                                 |
| Ask          | Explain codebase, answer questions                           |
| Plan         | Plan and design before implementation                        |
| Advanced     | More powerful work, including skills and broader tool access |
| Orchestrator | Coordinate complex tasks across specialized modes            |

IBM says skills are only available in **Advanced mode**. ([IBM Bob][6])

## Custom mode file

Project modes live in:

```text
.bob/custom_modes.yaml
```

Global modes live in:

```text
custom_modes.yaml
```

and can be edited through Bob settings. IBM says project modes are edited through `.bob/custom_modes.yaml`, and YAML is the preferred format. ([IBM Bob][7])

Example:

```yaml
customModes:
  - slug: docs-writer
    name: Documentation Writer
    roleDefinition: >
      You are a technical documentation architect specializing in clear,
      maintainable engineering documentation.
    whenToUse: >
      Use this mode for README updates, architecture docs, API docs,
      onboarding docs, and documentation gap analysis.
    customInstructions: >
      Focus on clarity, structure, accuracy, and maintainability.
      Do not change source code unless explicitly asked.
    groups:
      - read
      - - edit
        - fileRegex: \.(md|mdx)$
          description: Markdown files only
      - browser
```

Bob custom modes support the following tool groups:

```text
read       = read files and directories
edit       = modify files, optionally restricted with fileRegex
browser    = browser automation
command    = terminal commands
mcp        = MCP servers
skill      = skills
```

IBM documents these tool groups and the ability to restrict the edit group with `fileRegex`. ([IBM Bob][7])

## Recommended custom modes for a software repo

### 5.1 Read-only architect mode

```yaml
customModes:
  - slug: architect-reviewer
    name: Architect Reviewer
    roleDefinition: >
      You are a senior software architect. You analyze architecture,
      module boundaries, dependencies, scalability, maintainability,
      and operational risk.
    whenToUse: >
      Use before major features, refactors, or architectural decisions.
    customInstructions: >
      Do not edit files. Return options, trade-offs, risks, and a recommended path.
    groups:
      - read
```

### 5.2 Docs-only mode

```yaml
customModes:
  - slug: docs-writer
    name: Documentation Writer
    roleDefinition: >
      You are a technical writer and documentation architect.
    whenToUse: >
      Use for README, ADR, API, onboarding, and operational documentation.
    customInstructions: >
      Only edit documentation files. Preserve technical accuracy.
    groups:
      - read
      - - edit
        - fileRegex: \.(md|mdx|txt)$
          description: Documentation files only
```

### 5.3 Test engineer mode

```yaml
customModes:
  - slug: test-engineer
    name: Test Engineer
    roleDefinition: >
      You are a test engineer focused on robust, maintainable test coverage.
    whenToUse: >
      Use when adding tests, debugging failing tests, or reviewing coverage gaps.
    customInstructions: >
      Reuse existing test patterns. Prefer focused tests before broad integration tests.
    groups:
      - read
      - - edit
        - fileRegex: .*(test|spec).*\.(js|jsx|ts|tsx|py|java|go)$
          description: Test files only
      - command
```

### 5.4 Security reviewer mode

```yaml
customModes:
  - slug: security-reviewer
    name: Security Reviewer
    roleDefinition: >
      You are a security reviewer focused on secrets, authentication,
      authorization, dependency risk, infrastructure risk, and data exposure.
    whenToUse: >
      Use for auth, IAM, infrastructure, dependency, input validation,
      logging, and data-handling changes.
    customInstructions: >
      Do not edit files. Report severity, evidence, exploitability, and remediation.
    groups:
      - read
```

## Overriding built-in modes

Bob lets you override default modes by creating a custom mode with the same slug in `.bob/custom_modes.yaml`. IBM gives the example of overriding `code` to make Code mode Python-only with edit restrictions. Project-specific overrides take precedence over global overrides, which take precedence over defaults. ([IBM Bob][7])

---

# 6. Mode-specific rules

Bob supports mode-specific rules using either directory structure or single files.

Preferred:

```text
.bob/
└── rules-{mode-slug}/
    ├── 01-style-guide.md
    └── 02-formatting.txt
```

Alternative:

```text
.bobrules-{mode-slug}
```

IBM says the directory method takes precedence if both exist, files are loaded alphabetically, and they are combined with the `customInstructions` field from the mode configuration. ([IBM Bob][7])

Example:

```text
.bob/
├── rules-docs-writer/
│   ├── 01-doc-style.md
│   └── 02-api-docs.md
├── rules-test-engineer/
│   └── 01-test-policy.md
└── rules-security-reviewer/
    └── 01-security-review.md
```

---

# 7. Skills

Skills are reusable instruction sets for specialized workflows. IBM describes them as “recipes” Bob follows to complete specific work consistently. Skills can include supporting files like checklists, templates, reference docs, scripts, and utilities. ([IBM Bob][6])

Important details:

```text
Project skills: <project>/.bob/skills/
Global skills:  ~/.bob/skills/
```

If both locations contain a skill with the same name, the project-level skill takes precedence. ([IBM Bob][6])

Skills are only available in **Advanced mode**. ([IBM Bob][6])

By default, Bob asks for permission before activating a skill. You can skip this prompt by enabling Skills in Auto-Approve, but IBM classifies Skills auto-approval as medium risk. ([IBM Bob][6])

## Skill structure

```text
.bob/
└── skills/
    └── code-review/
        ├── SKILL.md
        ├── checklist.md
        ├── severity-guide.md
        └── scripts/
            └── analyze.sh
```

IBM says the `SKILL.md` file uses YAML frontmatter followed by skill instructions. Required fields are `name` and `description`; skills without descriptions are ignored. Everything below the frontmatter becomes the instructions Bob receives when the skill activates. ([IBM Bob][6])

## Skill template

```md
---
name: review-pr
description: Review local changes or a branch for correctness, security, maintainability, performance, and missing tests.
---

# Review PR skill

Review the current change like a senior maintainer.

<Steps>
<Step>
Inspect the changed files and understand the intended behavior.
</Step>

<Step>
Review correctness risks first.
</Step>

<Step>
Review security, maintainability, performance, and test coverage.
</Step>

<Step>
Suggest minimal fixes before suggesting larger refactors.
</Step>

<Step>
Return findings with severity and concrete evidence.
</Step>
</Steps>

Use `checklist.md` and `severity-guide.md` when classifying findings.

Output format:

1. Summary
2. Critical findings
3. High-risk findings
4. Medium-risk findings
5. Test gaps
6. Suggested next actions
7. Confidence level
```

## Recommended skills

### `implement-feature`

```text
.bob/skills/implement-feature/SKILL.md
```

```md
---
name: implement-feature
description: Implement a feature using existing architecture, minimal diffs, relevant tests, and a final risk summary.
---

# Implement feature

<Steps>
<Step>
Inspect existing project patterns before editing.
</Step>

<Step>
Create a short implementation plan.
</Step>

<Step>
Make the smallest useful change.
</Step>

<Step>
Add or update tests.
</Step>

<Step>
Run the smallest relevant test.
</Step>

<Step>
Summarize changed files, tests run, and residual risks.
</Step>
</Steps>

Rules:

- Do not introduce new dependencies unless necessary.
- Do not change public APIs without explaining why.
- Do not commit or push unless explicitly asked.
```

### `debug-failing-test`

```md
---
name: debug-failing-test
description: Debug a failing test or runtime error by finding the root cause, proposing a minimal fix, and validating with the narrowest relevant test.
---

# Debug failing test

<Steps>
<Step>
Identify the exact failure, stack trace, or failing assertion.
</Step>

<Step>
Trace the relevant code path.
</Step>

<Step>
Explain the likely root cause.
</Step>

<Step>
Apply the smallest safe fix.
</Step>

<Step>
Run the narrowest relevant test.
</Step>

<Step>
Report confidence and remaining risks.
</Step>
</Steps>
```

### `write-adr`

```md
---
name: write-adr
description: Create an Architecture Decision Record with context, options, trade-offs, decision, consequences, and revisit criteria.
---

# Write ADR

Create or update an ADR for the requested decision.

Use this structure:

1. Title
2. Status
3. Context
4. Decision
5. Options considered
6. Trade-offs
7. Consequences
8. Rollback or revisit criteria

Use `adr-template.md` if present.
```

### `audit-terraform-change`

```md
---
name: audit-terraform-change
description: Review Terraform or infrastructure-as-code changes for safety, blast radius, security risk, drift risk, and rollback strategy.
---

# Audit Terraform change

Do not run `terraform apply`.
Do not run `terraform destroy`.
Do not approve production changes.

Review:

- IAM changes
- Network/security group changes
- Secret references
- State assumptions
- Environment overlays
- Provider changes
- Resource replacement risks

Output:

1. Summary
2. Blast radius
3. Security risks
4. Operational risks
5. Rollback strategy
6. Required manual validation
```

IBM recommends clear skill descriptions, focused `SKILL.md` files, moving detailed reference material into supporting files, and structuring instructions as actionable steps. ([IBM Bob][6])

---

# 8. Slash commands

Bob supports custom slash commands through markdown files. Project commands live in `.bob/commands/`; global commands live in `~/.bob/commands/`. The filename becomes the command name. For example, `test-api.md` becomes `/test-api`. ([IBM Bob][8])

Bob also has built-in commands such as:

```text
/init
/review
/create-pr
```

The `/review` command can review uncommitted changes, compare against branches, and validate issue coverage. IBM says it performs bug detection, security checks, performance checks, and style consistency analysis. ([IBM Bob][8])

## Custom command example

```text
.bob/commands/release-check.md
```

```md
---
description: Perform a release readiness check
argument-hint: <target-version>
---

Perform a release readiness review for version $1.

Check:

1. Changelog
2. Version bump
3. Tests
4. Dependency changes
5. Database migrations
6. Infrastructure changes
7. Rollback notes
8. Known risks

Return a concise release risk report.
```

Slash command frontmatter supports:

```text
description
argument-hint
```

IBM documents positional arguments like `$1` and `$2` in custom command markdown. ([IBM Bob][8])

## Skill vs command

Use a **command** when you want a quick, user-invoked prompt shortcut.

Use a **skill** when you want a reusable workflow with activation metadata, supporting files, checklists, scripts, and automatic selection.

Example:

```text
.bob/commands/review-branch.md       = quick manual shortcut
.bob/skills/review-pr/SKILL.md       = reusable structured workflow
```

---

# 9. MCP configuration

Bob supports MCP servers globally and project-locally. IBM’s MCP tutorial says MCP extends Bob by connecting external tools and services, with STDIO for local servers and SSE for remote HTTP/HTTPS servers. It also says users can configure servers using JSON files that define commands, environment variables, and auto-approved tools. ([IBM][9])

Important paths:

```text
Global MCP:  mcp_settings.json
Project MCP: .Bob/mcp.json
```

IBM’s tutorial uses `.Bob/mcp.json` with uppercase `B`; most Bob project files in the docs use `.bob/`. Because IBM’s MCP tutorial explicitly states `.Bob/mcp.json`, follow the IDE-generated path for your installed Bob version, and be consistent in your repo. ([IBM][9])

Example MCP config from IBM’s documented shape:

```json
{
  "mcpServers": {
    "arxiv": {
      "command": "node",
      "args": ["/absolute/path/to/arxiv-server/build/index.js"],
      "disabled": false,
      "alwaysAllow": [],
      "disabledTools": []
    }
  }
}
```

IBM says project-level MCP settings override global ones and are useful for team sharing through version control. ([IBM][9])

## Practical recommendation

Use MCP for:

```text
- GitHub or GitLab operations
- Internal documentation search
- API testing tools
- Design system lookup
- Local database read-only tools
- Ticketing systems
- Cloud/Kubernetes observability tools
```

Do not use MCP as a replacement for Bob rules or skills. MCP gives tools. Skills give workflows. Rules give behavioral constraints. Modes give personas and tool access.

---

# 10. Auto-approve: Bob’s closest equivalent to command permissions

Bob does not appear, from public docs, to expose a Claude/Codex-style repo-level hook system such as `PreToolUse` or `PostToolUse`. Instead, public Bob docs emphasize:

```text
- auto-approve toggles
- command-specific approval
- LLM risk detection
- AST-based command validation
- .bobignore
- custom mode tool restrictions
- MCP tool auto-approval
```

IBM warns that auto-approval bypasses confirmation prompts and can cause data loss, file corruption, or security compromise, especially command-line access. ([IBM Bob][10])

Bob auto-approve categories include:

```text
Read
Write
Browser
Retry
MCP
Mode
Subtasks
Execute
Question
Todo
Skills
```

IBM labels Write, Browser, and Execute as high risk, MCP as medium-high risk, and Skills as medium risk. ([IBM Bob][10])

## Execute approval

Bob uses a two-stage approval process for terminal execution:

1. Enable Execute auto-approve.
2. Approve individual command patterns.

Future commands matching approved patterns can run automatically. Bob also includes LLM risk detection and AST-based command validation to detect risky commands and command chaining such as `&&`, `||`, `;`, or `|`. ([IBM Bob][10])

## Recommended auto-approve policy

For serious repos:

```text
Safe to consider:
- Read
- Mode
- Todo
- Retry

Use carefully:
- Skills
- MCP
- Subtasks

Usually keep manual:
- Write
- Browser
- Execute
- Question
```

For enterprise/cloud repos, I would keep **Execute disabled by default** and approve commands case by case.

---

# 11. Pre-hooks and post-hooks

I did not find public IBM Bob documentation for a general project-level hook mechanism equivalent to Claude Code hooks or Codex hooks.

So the honest answer is:

```text
IBM Bob public docs do not currently describe repo-level PreToolUse/PostToolUse hooks.
Use Bob’s supported mechanisms instead:
- .bobignore
- auto-approve controls
- command-specific approval
- custom modes with restricted tool groups
- rules
- skills
- MCP disabledTools / alwaysAllow
- slash commands
- normal project scripts and CI
```

## Practical equivalents

| Need                      | Claude/Codex hook style | IBM Bob practical equivalent                                                       |
| ------------------------- | ----------------------- | ---------------------------------------------------------------------------------- |
| Block reading secrets     | PreToolUse hook         | `.bobignore` plus no risky auto-approve                                            |
| Restrict editing          | Tool permission hook    | custom mode `edit` with `fileRegex`                                                |
| Run formatter after edits | PostToolUse hook        | skill instruction, slash command, package scripts, CI                              |
| Block dangerous shell     | PreToolUse hook         | keep Execute manual, command-specific approval, LLM risk detection, AST validation |
| Validate final output     | Stop hook               | skill checklist or slash command                                                   |
| Add startup context       | SessionStart hook       | `AGENTS.md`, `.bob/rules/`, mode rules                                             |
| Connect external tools    | MCP hook/tool           | `.Bob/mcp.json` / MCP settings                                                     |

For example, instead of a post-edit hook that auto-runs tests, create a skill:

```text
.bob/skills/implement-feature/SKILL.md
```

with an explicit final step:

```md
Before completion:
1. Run the smallest relevant test.
2. Run lint/typecheck if the project has commands for it.
3. If tests cannot run, explain why.
```

And enforce it through CI.

---

# 12. Subagents, subtasks, and Orchestrator

IBM Bob public docs use **Subtasks** and **Orchestrator mode**, not a public custom subagent file structure like Claude’s `.claude/agents/*.md` or Codex’s `.codex/agents/*.toml`.

Bob’s Auto-approve docs define Subtasks as separate task instances that Bob creates to break complex work into manageable pieces. ([IBM Bob][10])

Bob’s Modes docs describe Orchestrator mode as a strategic workflow orchestrator that coordinates complex tasks by delegating them to appropriate specialized modes. Orchestrator has no direct tool access according to the mode table snippet, which is consistent with its coordination role. ([IBM Bob][5])

So in Bob:

```text
Custom mode      = specialized persona + tool access
Orchestrator     = coordinator across modes
Subtask          = Bob-created task instance for decomposed work
Skill            = reusable workflow recipe
```

## How to model “subagents” in Bob

Since Bob does not publicly document custom subagent files, model subagents using **custom modes** plus Orchestrator:

```yaml
customModes:
  - slug: security-reviewer
    name: Security Reviewer
    roleDefinition: >
      You are a security reviewer focused on secrets, auth, authorization,
      dependency risk, input validation, and data exposure.
    whenToUse: >
      Use for security-sensitive code, infrastructure, IAM, and dependency changes.
    customInstructions: >
      Do not edit files. Return severity, evidence, exploitability, and remediation.
    groups:
      - read

  - slug: test-engineer
    name: Test Engineer
    roleDefinition: >
      You are a test engineer focused on robust, maintainable test coverage.
    whenToUse: >
      Use for new tests, failing tests, regression checks, and coverage gaps.
    customInstructions: >
      Reuse existing test patterns and prefer focused tests.
    groups:
      - read
      - - edit
        - fileRegex: .*(test|spec).*\.(js|jsx|ts|tsx|py|java|go)$
          description: Test files only
      - command
```

Then ask Bob in Orchestrator mode:

```text
Use Orchestrator mode. Break this PR review into subtasks:
1. correctness review
2. security review
3. test coverage review
4. DevOps/operational risk review

Use the relevant specialized modes where appropriate. Do not edit files. Return a consolidated risk report.
```

That is Bob’s closest public equivalent to a multi-subagent workflow.

---

# 13. Plugins

I did not find public IBM Bob documentation for a general “plugin” packaging system equivalent to Claude Code plugins or Codex plugins.

What Bob publicly documents instead is:

```text
- MCP servers and Bob Marketplace for external tools
- custom modes
- custom rules
- skills
- slash commands
```

IBM’s MCP tutorial says Bob lets you explore community servers through the Bob Marketplace or build your own with the MCP SDK. It also says MCP servers can be globally or project configured and can define commands, environment variables, and auto-approved tools. ([IBM][9])

So for Bob, treat “plugin” as one of two things:

1. **MCP server**: if you need external tools or services.
2. **Reusable `.bob/` package**: if you want to copy shared modes, skills, rules, and commands across repos.

## Practical Bob “plugin-like” package

```text
bob-engineering-kit/
├── rules/
│   ├── 01-engineering-standards.md
│   └── 02-security.md
├── custom_modes.yaml
├── skills/
│   ├── review-pr/
│   │   └── SKILL.md
│   └── write-adr/
│       └── SKILL.md
└── commands/
    ├── release-check.md
    └── explain-module.md
```

Install manually into a repo:

```bash
mkdir -p .bob
cp -R bob-engineering-kit/rules .bob/
cp -R bob-engineering-kit/skills .bob/
cp -R bob-engineering-kit/commands .bob/
cp bob-engineering-kit/custom_modes.yaml .bob/custom_modes.yaml
```

For external tools, build an MCP server and add it to `.Bob/mcp.json` or through the Bob MCP settings UI.

---

# 14. Workflow

In Bob terminology, a workflow can mean:

```text
- a natural-language task process
- a slash command
- a skill
- Orchestrator-driven decomposition
- MCP-backed automation
```

There is no single repo artifact named `workflow` in the public docs I found. For repo design, model workflows as:

| Workflow type                 | Bob artifact                                       |
| ----------------------------- | -------------------------------------------------- |
| Simple reusable prompt        | `.bob/commands/*.md`                               |
| Multi-step repeatable process | `.bob/skills/<name>/SKILL.md`                      |
| Role-specific process         | `.bob/custom_modes.yaml` plus `.bob/rules-{mode}/` |
| External-system automation    | `.Bob/mcp.json` plus MCP server                    |
| Complex multi-specialist task | Orchestrator mode plus subtasks                    |

Example workflow stack for PR review:

```text
AGENTS.md
.bob/rules/02-testing.md
.bob/rules/03-security.md
.bob/custom_modes.yaml
.bob/skills/review-pr/SKILL.md
.bob/commands/review-branch.md
```

Then invoke:

```text
/review-branch main
```

or:

```text
Use the review-pr skill to review this branch against main.
```

---

# 15. Full recommended IBM Bob repo setup

For your kind of enterprise engineering, cloud, AI-agent, React, DevOps, and documentation projects, I would use this:

```text
repo/
├── AGENTS.md
├── .bobignore
├── .bob/
│   ├── custom_modes.yaml
│   ├── rules/
│   │   ├── 01-project-standards.md
│   │   ├── 02-testing.md
│   │   ├── 03-security.md
│   │   └── 04-git-and-pr.md
│   ├── rules-code/
│   │   └── 01-code-mode.md
│   ├── rules-plan/
│   │   └── 01-planning.md
│   ├── rules-docs-writer/
│   │   └── 01-documentation.md
│   ├── rules-security-reviewer/
│   │   └── 01-security-review.md
│   ├── skills/
│   │   ├── implement-feature/
│   │   │   └── SKILL.md
│   │   ├── review-pr/
│   │   │   ├── SKILL.md
│   │   │   ├── checklist.md
│   │   │   └── severity-guide.md
│   │   ├── debug-failing-test/
│   │   │   └── SKILL.md
│   │   ├── write-adr/
│   │   │   ├── SKILL.md
│   │   │   └── adr-template.md
│   │   ├── audit-terraform-change/
│   │   │   └── SKILL.md
│   │   └── release-check/
│   │       └── SKILL.md
│   ├── commands/
│   │   ├── review-branch.md
│   │   ├── release-check.md
│   │   ├── explain-module.md
│   │   └── write-pr-description.md
│   └── mcp.json
└── docs/
    ├── architecture.md
    ├── testing.md
    ├── release-process.md
    └── security.md
```

---

# 16. Decision table: custom mode vs workflow vs skill vs plugin vs subagent

| Concept             | Bob artifact                                   | Use when                                           | Example                               |
| ------------------- | ---------------------------------------------- | -------------------------------------------------- | ------------------------------------- |
| Project memory      | `AGENTS.md`                                    | Always-relevant project context                    | repo map, commands, standards         |
| Rule                | `.bob/rules/*.md`                              | Behavior constraints and standards                 | coding style, test policy             |
| Custom mode         | `.bob/custom_modes.yaml`                       | Specialized persona and tool access                | docs-only writer, security reviewer   |
| Mode-specific rules | `.bob/rules-{mode}/`                           | Extra instructions for one mode                    | security review checklist             |
| Skill               | `.bob/skills/<name>/SKILL.md`                  | Repeatable workflow with optional supporting files | PR review, ADR writing                |
| Slash command       | `.bob/commands/*.md`                           | Quick manual shortcut                              | `/release-check`                      |
| MCP tool            | `.Bob/mcp.json` / MCP settings                 | External tool/service access                       | GitHub, docs, database, APIs          |
| Plugin              | Not clearly public as first-class Bob artifact | Treat as MCP server or reusable `.bob/` package    | internal Bob engineering kit          |
| Subagent            | Not public as custom file structure            | Use custom modes + Orchestrator + subtasks         | security-reviewer mode                |
| Hook                | Not publicly documented as project hook system | Use supported controls instead                     | `.bobignore`, auto-approve, modes, CI |

---

# 17. Bob vs Claude Code vs Codex

| Area                     | IBM Bob                              | Claude Code                           | OpenAI Codex                |
| ------------------------ | ------------------------------------ | ------------------------------------- | --------------------------- |
| Main project instruction | `AGENTS.md`                          | `CLAUDE.md`                           | `AGENTS.md`                 |
| Project config folder    | `.bob/`                              | `.claude/`                            | `.codex/` and `.agents/`    |
| Rules                    | `.bob/rules/`, `.bobrules`           | `CLAUDE.md`, settings, hooks          | `AGENTS.md`, `.codex/rules` |
| Skills                   | `.bob/skills/`                       | `.claude/skills/`                     | `.agents/skills/`           |
| Custom modes             | `.bob/custom_modes.yaml`             | Not same concept                      | Not same concept            |
| Subagents                | Orchestrator + subtasks + modes      | `.claude/agents/*.md`                 | `.codex/agents/*.toml`      |
| Hooks                    | Not found in public docs             | First-class hooks                     | First-class hooks           |
| Plugins                  | Not found as generic public artifact | First-class plugin system             | First-class plugin system   |
| External tools           | MCP settings / `.Bob/mcp.json`       | MCP                                   | MCP                         |
| File ignore              | `.bobignore`                         | permissions / ignores depending setup | sandbox/rules/config        |

---

# 18. Recommended rollout plan

## Phase 1: Minimum Bob-ready repo

Add:

```text
AGENTS.md
.bobignore
.bob/rules/01-project-standards.md
.bob/rules/02-testing.md
.bob/rules/03-security.md
```

This gives Bob project context, behavioral guidance, and file boundaries.

## Phase 2: Add safe modes

Add:

```text
.bob/custom_modes.yaml
.bob/rules-docs-writer/
.bob/rules-security-reviewer/
.bob/rules-test-engineer/
```

Start with read-only or restricted-edit modes.

## Phase 3: Add skills

Add:

```text
.bob/skills/review-pr/
.bob/skills/implement-feature/
.bob/skills/debug-failing-test/
.bob/skills/write-adr/
```

Use skills for workflows you repeat often.

## Phase 4: Add slash commands

Add:

```text
.bob/commands/review-branch.md
.bob/commands/release-check.md
.bob/commands/explain-module.md
```

Use commands for quick manual invocation.

## Phase 5: Add MCP only when needed

Add:

```text
.Bob/mcp.json
```

or configure MCP through Bob’s settings UI. Use project-level MCP when the tool is team-relevant and safe to share. Use global MCP for personal tools.

---

# 19. Recommended default for your use case

For your project repos, I would use this default:

```text
AGENTS.md
.bobignore
.bob/rules/
.bob/custom_modes.yaml
.bob/skills/
.bob/commands/
```

I would not begin with MCP or plugin-like packaging. Add MCP only when the repo genuinely needs external tools. Package a reusable “Bob engineering kit” only after the same rules, modes, commands, and skills prove useful across multiple repos.

The most useful starting set:

```text
AGENTS.md
.bobignore
.bob/rules/01-project-standards.md
.bob/rules/02-testing.md
.bob/rules/03-security.md
.bob/custom_modes.yaml
.bob/skills/review-pr/SKILL.md
.bob/skills/implement-feature/SKILL.md
.bob/skills/debug-failing-test/SKILL.md
.bob/commands/review-branch.md
```

That gives you a practical, version-controlled, team-shareable Bob setup without relying on undocumented hooks or plugin mechanisms.

[1]: https://bob.ibm.com/docs/ide "Welcome to IBM Bob | Docs | IBM Bob"
[2]: https://bob.ibm.com/docs/ide/configuration/rules "Custom rules | Docs | IBM Bob"
[3]: https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project?utm_source=chatgpt.com "Start a project with /init and AGENTS.md | Docs"
[4]: https://bob.ibm.com/docs/ide/configuration/bobignore "Ignoring files | Docs | IBM Bob"
[5]: https://bob.ibm.com/docs/ide/features/modes "Modes | Docs | IBM Bob"
[6]: https://bob.ibm.com/docs/ide/features/skills "Skills | Docs | IBM Bob"
[7]: https://bob.ibm.com/docs/ide/configuration/custom-modes "Custom modes | Docs | IBM Bob"
[8]: https://bob.ibm.com/docs/ide/features/slash-commands "Slash commands | Docs | IBM Bob"
[9]: https://www.ibm.com/think/tutorials/mcp-integration-ibm-bob "MCP Integration using IBM Bob "
[10]: https://bob.ibm.com/docs/ide/features/auto-approving-actions "Auto-approve | Docs | IBM Bob"
