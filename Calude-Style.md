Below is a research-backed, repo-oriented report for setting up a **Claude Code-ready project repository**, including `CLAUDE.md`, `.claude/settings.json`, hooks, skills, subagents, plugins, and when to use each.

## 1. Mental model: what Claude Code reads from a repo

Claude Code is not configured only by one file. A project can influence Claude through several layers:

```text
your-repo/
├── CLAUDE.md
├── CLAUDE.local.md                 # private, gitignored
├── .claude/
│   ├── settings.json               # shared project settings
│   ├── settings.local.json         # private project settings, gitignored
│   ├── agents/
│   │   └── code-reviewer.md
│   ├── skills/
│   │   └── deploy-staging/
│   │       ├── SKILL.md
│   │       ├── reference.md
│   │       └── scripts/
│   ├── rules/
│   │   └── frontend.md
│   └── commands/                   # older command format, still works
├── .mcp.json                       # project MCP servers
└── package.json / Makefile / README.md
```

Claude Code reads instructions, settings, skills, subagents, and memory from the project `.claude` directory and from the user’s home `~/.claude` directory. Project files can be committed to share with the team, while `~/.claude` is personal configuration. Anthropic’s docs explicitly say most users only need to edit `CLAUDE.md` and `settings.json`, then add skills, rules, or subagents as needed. ([Claude Code][1])

The important distinction is this: **`CLAUDE.md` is context, not enforcement**. Claude reads it at startup and tries to follow it, but if you need something to happen every time, such as blocking a destructive shell command or running a formatter after edits, use settings, permissions, and hooks. Anthropic’s memory docs state that `CLAUDE.md` and auto memory are loaded as context, while a `PreToolUse` hook should be used to block actions regardless of what Claude decides. ([Claude Code][2])

---

## 2. Recommended repo structure for a serious Claude-enabled project

For a professional software repo, I would use this structure:

```text
your-repo/
├── CLAUDE.md
├── .claude/
│   ├── settings.json
│   ├── rules/
│   │   ├── frontend.md
│   │   ├── backend.md
│   │   ├── tests.md
│   │   └── infrastructure.md
│   ├── agents/
│   │   ├── repo-architect.md
│   │   ├── code-reviewer.md
│   │   ├── test-engineer.md
│   │   ├── security-reviewer.md
│   │   └── devops-reviewer.md
│   ├── skills/
│   │   ├── implement-feature/
│   │   │   └── SKILL.md
│   │   ├── review-pr/
│   │   │   └── SKILL.md
│   │   ├── debug-failing-test/
│   │   │   └── SKILL.md
│   │   ├── create-doc/
│   │   │   ├── SKILL.md
│   │   │   └── template.md
│   │   └── release-check/
│   │       ├── SKILL.md
│   │       └── checklist.md
│   └── hooks/
│       ├── block-dangerous-commands.sh
│       ├── run-format-after-edit.sh
│       ├── run-tests-after-src-change.sh
│       └── validate-env-access.sh
├── .mcp.json
└── docs/
    ├── architecture.md
    ├── local-development.md
    ├── testing.md
    └── release-process.md
```

Keep the root `CLAUDE.md` short and stable. Put reusable workflows into skills. Put role-specific deep work into subagents. Put deterministic safety and quality gates into hooks. Put external tools and integrations into MCP or plugins. This separation matters because Claude Code loads `CLAUDE.md` at startup, while a skill body loads only when the skill is used, reducing recurring token cost. ([Claude Code][2])

---

## 3. What should be in `CLAUDE.md`

Use `CLAUDE.md` for **always-relevant project context**. Anthropic recommends adding content when Claude repeats the same mistake, when code review catches something Claude should have known, when you repeatedly type the same correction, or when a new teammate would need the same context. It should contain facts Claude should hold in every session: build commands, conventions, project layout, and “always do X” rules. Anthropic recommends targeting under 200 lines per `CLAUDE.md` for better adherence. ([Claude Code][2])

### Recommended `CLAUDE.md` template

````md
# Project Guide for Claude

## Project purpose

This repository contains [short description of product/system].
The main goal is to [business/user outcome].

## Tech stack

- Frontend: [React/Angular/Vue/etc.]
- Backend: [Node/FastAPI/Go/Java/etc.]
- Database: [PostgreSQL/DynamoDB/etc.]
- Infrastructure: [AWS/OpenShift/Kubernetes/Terraform/etc.]
- Tests: [Jest/Pytest/Go test/etc.]

## Repository map

- `src/`: application source code
- `src/components/`: UI components
- `src/services/`: API and business service logic
- `src/tests/`: test helpers and fixtures
- `infra/`: infrastructure-as-code
- `docs/`: architecture and operational documentation

## Local development

Use these commands:

```bash
npm install
npm run lint
npm test
npm run build
````

Do not invent commands. If a command is missing, inspect `package.json`, `Makefile`, `README.md`, or ask before adding a new workflow.

## Coding standards

* Prefer small, focused changes.
* Preserve existing architecture unless the user explicitly asks for refactoring.
* Do not change public APIs without calling it out.
* Do not remove tests unless they are obsolete and the reason is documented.
* Prefer existing project patterns over introducing new libraries.
* Avoid broad formatting-only diffs.

## Testing expectations

Before claiming a change is complete:

1. Run the smallest relevant test first.
2. Run lint/typecheck when the change touches typed code.
3. Run the broader suite only if the change is cross-cutting.
4. If tests cannot be run, explain exactly why.

## Security and secrets

* Never read or print `.env`, `.env.*`, credentials, private keys, or production secrets.
* Never hardcode secrets.
* Use existing secret management patterns.
* Treat migration scripts, infrastructure changes, and auth changes as high-risk.

## Git and PR behavior

* Do not commit unless explicitly asked.
* Keep diffs minimal.
* Summarize changed files and risks after each meaningful change.
* For PR descriptions, include: summary, tests, risks, rollback notes.

## Communication style

* Be precise.
* State assumptions.
* Explain trade-offs for architectural changes.
* Prefer practical recommendations over generic advice.

````

### What not to put in `CLAUDE.md`

Avoid large API references, long tutorials, long architectural documents, full style guides, or multi-step playbooks. Anthropic’s docs say that if an entry is a multi-step procedure or only matters for one part of the codebase, move it to a skill or path-scoped rule instead. :contentReference[oaicite:4]{index=4}

---

## 4. `CLAUDE.local.md`: private personal context

Use `CLAUDE.local.md` for personal preferences that should not be committed. Anthropic lists `CLAUDE.local.md` as a project-root file for private project preferences, loaded alongside `CLAUDE.md`, and recommends adding it to `.gitignore`. :contentReference[oaicite:5]{index=5}

Example:

```md
# Personal Claude Preferences

- Prefer minimal diffs unless I explicitly request refactoring.
- Explain architectural trade-offs before changing structure.
- When debugging tests, show the failing assertion and suspected root cause first.
- Do not run expensive integration tests unless I ask.
````

Suggested `.gitignore`:

```gitignore
CLAUDE.local.md
.claude/settings.local.json
```

---

## 5. `.claude/settings.json`: project configuration

`settings.json` is the official mechanism for configuring Claude Code behavior through hierarchical settings. User settings live at `~/.claude/settings.json`, shared project settings at `.claude/settings.json`, and local project settings at `.claude/settings.local.json`. Managed settings can be deployed by organizations and cannot be overridden by user or project settings. ([Claude Code][3])

A strong project baseline:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm test *)",
      "Bash(npm run test *)",
      "Bash(npm run build)",
      "Bash(git diff *)",
      "Bash(git status *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Bash(rm -rf /)",
      "Bash(curl *)",
      "Bash(wget *)",
      "Bash(npm publish *)",
      "Bash(terraform apply *)",
      "Bash(kubectl delete *)",
      "Bash(helm uninstall *)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "0"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-dangerous-commands.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/run-format-after-edit.sh"
          }
        ]
      }
    ]
  }
}
```

The `$schema` entry enables editor autocomplete and validation. Claude Code watches settings files and reloads most changes during a running session, including `permissions` and `hooks`. Some fields, such as `model` and `outputStyle`, are read at session start or require `/clear` or restart. ([Claude Code][3])

### Settings precedence

Claude Code settings are hierarchical. Managed settings have the highest priority and cannot be overridden. Temporary `--settings` comes next, then `.claude/settings.local.json`, then `.claude/settings.json`, then `~/.claude/settings.json`. Array settings such as permission rules are concatenated and deduplicated rather than replaced. ([Claude Code][3])

### Sensitive file exclusion

To prevent Claude from accessing secrets, use `permissions.deny`. Anthropic explicitly says this replaces the deprecated `ignorePatterns` configuration, and denied files are excluded from file discovery/search and blocked from read operations. ([Claude Code][3])

Recommended deny list:

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./build)",
      "Read(./dist)",
      "Read(./node_modules)"
    ]
  }
}
```

---

## 6. Hooks: pre-hooks, post-hooks, and lifecycle automation

Hooks are deterministic automation points. Anthropic defines hooks as user-defined shell commands, HTTP endpoints, or LLM prompts that execute automatically at specific points in Claude Code’s lifecycle. Hook input is passed as JSON, and handlers can inspect the event, take action, and optionally return a decision. ([Claude Code][4])

Hooks are configured in three layers: choose the event, add a matcher group, then define one or more handlers. Hook locations include `~/.claude/settings.json`, `.claude/settings.json`, `.claude/settings.local.json`, managed policy settings, plugin `hooks/hooks.json`, and skill or agent frontmatter. ([Claude Code][4])

### Common hook events

Important events include:

| Event                        | Use case                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| `SessionStart`               | Show repo reminder, load environment info, validate setup    |
| `UserPromptSubmit`           | Validate or transform user prompt before Claude processes it |
| `PreToolUse`                 | Block unsafe commands, validate file access, enforce policy  |
| `PostToolUse`                | Run formatter, log tool use, inspect generated files         |
| `PostToolUseFailure`         | Capture failed commands, suggest recovery                    |
| `SubagentStart`              | Log or restrict subagent usage                               |
| `SubagentStop`               | Summarize subagent output or run verification                |
| `Stop`                       | Enforce final checklist before Claude stops                  |
| `ConfigChange`               | Validate settings changes                                    |
| `FileChanged`                | Trigger checks when watched files change                     |
| `PreCompact` / `PostCompact` | Preserve context before/after compaction                     |
| `SessionEnd`                 | Clean up temp files or write session log                     |

The docs list many lifecycle events, including `SessionStart`, `Setup`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`, `Stop`, `ConfigChange`, `FileChanged`, `WorktreeCreate`, `WorktreeRemove`, `PreCompact`, `PostCompact`, and `SessionEnd`. ([Claude Code][4])

### Pre-hook example: block dangerous commands

`.claude/hooks/block-dangerous-commands.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT="$(cat)"
COMMAND="$(echo "$INPUT" | jq -r '.tool_input.command // empty')"

if echo "$COMMAND" | grep -Eiq 'rm -rf /|rm -rf \*|kubectl delete|terraform apply|terraform destroy|npm publish|docker system prune|helm uninstall'; then
  jq -n --arg reason "Blocked dangerous command: $COMMAND" '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: $reason
    }
  }'
  exit 0
fi

exit 0
```

In Anthropic’s example, a `PreToolUse` hook can return JSON with `permissionDecision: "deny"` and a reason, causing Claude Code to block the tool call and show Claude the reason. A silent exit code 0 does not approve the action; it simply lets the normal permission flow continue. ([Claude Code][4])

### Post-hook example: run formatter after edits

`.claude/hooks/run-format-after-edit.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

if [ -f package.json ]; then
  if jq -e '.scripts.format' package.json >/dev/null 2>&1; then
    npm run format
  elif jq -e '.scripts.lint' package.json >/dev/null 2>&1; then
    npm run lint -- --fix || true
  fi
fi
```

Use `PostToolUse` for deterministic cleanup after successful tool calls. For example, a plugin hook can run after `Write|Edit` using the `PostToolUse` event, which is exactly the kind of event Anthropic shows in plugin hook examples. ([Claude Code][5])

### Hook matcher rules

Matchers filter when a hook runs. For `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, and `PermissionDenied`, the matcher filters on tool name. `Bash` matches Bash exactly, `Edit|Write` matches either tool, and regex-like matchers such as `mcp__.*` can match MCP tools. ([Claude Code][4])

---

## 7. Skills: reusable procedures and reference knowledge

A **skill** is a reusable instruction package. Claude Code skills are directories with a `SKILL.md` entry point, optional frontmatter, and optional supporting files. Claude can invoke them automatically when relevant, or the user can invoke them directly with `/skill-name`. Anthropic says skills are useful when you keep pasting the same instructions, checklist, or multi-step procedure into chat, or when part of `CLAUDE.md` has grown into a procedure rather than a fact. ([Claude Code][6])

### Skill locations

| Scope      | Path                                     |
| ---------- | ---------------------------------------- |
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md` |
| Project    | `.claude/skills/<skill-name>/SKILL.md`   |
| Plugin     | `<plugin>/skills/<skill-name>/SKILL.md`  |
| Enterprise | managed settings                         |

Anthropic documents that personal skills apply to all projects, project skills apply to the current project, and plugin skills apply where the plugin is enabled. When names collide, enterprise overrides personal, personal overrides project, and plugin skills are namespaced. ([Claude Code][6])

### Basic skill template

`.claude/skills/review-pr/SKILL.md`

````md
---
name: review-pr
description: Review the current branch or diff for correctness, security, maintainability, and missing tests. Use when the user asks for PR review, code review, risk review, or pre-merge validation.
argument-hint: "[base-branch]"
allowed-tools: Read, Grep, Glob, Bash
---

## Goal

Review the current changes against `$ARGUMENTS`.

If no base branch is provided, use `main`.

## Steps

1. Inspect the current branch and changed files.
2. Read only files relevant to the change.
3. Identify correctness risks, security risks, maintainability issues, and missing tests.
4. Suggest minimal changes first.
5. Do not modify files unless the user explicitly asks.

## Commands

Use:

```bash
git diff $ARGUMENTS...HEAD
git status --short
````

## Output format

Return:

1. Summary
2. High-risk issues
3. Medium-risk issues
4. Test gaps
5. Suggested next actions

````

### Skill frontmatter fields

Important skill frontmatter fields include `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `model`, `paths`, and `shell`. Anthropic recommends `description` so Claude knows when to use the skill, and warns that skill descriptions plus `when_to_use` are truncated at 1,536 characters in the skill listing to reduce context usage. :contentReference[oaicite:18]{index=18}

### When to disable automatic skill invocation

Use:

```yaml
disable-model-invocation: true
````

for workflows with side effects, such as deployment, publishing, release creation, or sending messages. Anthropic explicitly recommends this for workflows like `/commit`, `/deploy`, or `/send-slack-message`, because you do not want Claude deciding on its own that a deployment should happen. ([Claude Code][6])

Example:

```md
---
name: deploy-production
description: Deploy production after explicit user approval.
disable-model-invocation: true
allowed-tools: Bash
---

Only run this skill when the user explicitly invokes `/deploy-production`.

Before deploying:
1. Confirm branch.
2. Confirm clean working tree.
3. Confirm tests passed.
4. Ask for explicit final approval.
```

### Supporting files inside skills

A skill can include supporting files such as `reference.md`, `examples.md`, templates, and scripts. Anthropic recommends keeping `SKILL.md` focused and referencing supporting files so Claude knows when to load them. It also recommends keeping `SKILL.md` under 500 lines and moving detailed references to separate files. ([Claude Code][6])

Example:

```text
.claude/skills/create-architecture-doc/
├── SKILL.md
├── template.md
├── examples/
│   └── architecture-decision-record.md
└── scripts/
    └── validate-links.sh
```

---

## 8. Subagents: specialized workers with their own context

A **subagent** is a specialized AI assistant that runs in its own context window with a custom system prompt, specific tool access, and independent permissions. Use a subagent when a side task would flood the main conversation with logs, search results, or file contents that you do not want in the main context. Claude delegates to a subagent based on the subagent’s `description`. ([Claude API Docs][7])

### Subagent locations

| Scope       | Path / mechanism      |
| ----------- | --------------------- |
| Managed     | managed settings      |
| CLI session | `claude --agents ...` |
| Project     | `.claude/agents/`     |
| User        | `~/.claude/agents/`   |
| Plugin      | `<plugin>/agents/`    |

Project subagents are ideal for codebase-specific workers and should be checked into version control when useful for the team. User subagents are personal and apply across projects. Claude Code scans project and user agent directories recursively, but identity comes from the `name` frontmatter, not the filename. If duplicate names exist within one scope, Claude Code keeps one and discards the other without warning. ([Claude Code][8])

### Subagent template

`.claude/agents/code-reviewer.md`

```md
---
name: code-reviewer
description: Reviews code changes for correctness, maintainability, security, and missing tests. Use after implementation or before creating a pull request.
tools: Read, Glob, Grep, Bash
model: sonnet
maxTurns: 20
memory: project
color: blue
---

You are a senior code reviewer for this repository.

Focus on:
- Correctness
- Security
- Maintainability
- Test coverage
- Backward compatibility
- Consistency with existing project patterns

Do not edit files unless the user explicitly asks.
Prefer concrete findings over generic advice.

Review process:
1. Inspect changed files.
2. Read relevant surrounding code.
3. Identify high-risk issues first.
4. Suggest minimal fixes.
5. State what tests should be run.

Output:
- Summary
- Critical issues
- Suggested improvements
- Missing tests
- Confidence level
```

### Subagent frontmatter fields

Only `name` and `description` are required. Other supported fields include `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `color`, and `initialPrompt`. `isolation: worktree` gives the subagent an isolated temporary git worktree. ([Claude Code][8])

### Useful subagents for a software repo

#### 1. Repo architect

```md
---
name: repo-architect
description: Analyzes repository structure, module boundaries, dependencies, and architectural trade-offs. Use before large refactors or feature design.
tools: Read, Glob, Grep, Bash
model: sonnet
maxTurns: 25
memory: project
color: purple
---

You are a software architect reviewing this repository.

Analyze:
- Current architecture
- Module boundaries
- Dependency direction
- Scalability concerns
- Maintainability risks
- Operational risks

Do not edit files. Return options, trade-offs, and a recommended default.
```

#### 2. Test engineer

```md
---
name: test-engineer
description: Designs or reviews tests for a code change. Use when implementing features, fixing bugs, or improving coverage.
tools: Read, Glob, Grep, Bash, Edit, Write
model: sonnet
maxTurns: 20
memory: project
color: green
---

You are a test engineer.

Your job:
- Identify missing test cases
- Add minimal tests using existing project patterns
- Prefer focused unit tests before broad integration tests
- Avoid rewriting unrelated tests
- Run the smallest relevant test command first
```

#### 3. Security reviewer

```md
---
name: security-reviewer
description: Reviews auth, secrets, input validation, dependency, infrastructure, and data exposure risks. Use for security-sensitive changes.
tools: Read, Glob, Grep, Bash
model: sonnet
maxTurns: 20
memory: project
color: red
---

You are a security reviewer.

Focus on:
- Secrets exposure
- Injection risks
- Auth and authorization logic
- Unsafe shell commands
- Dependency risk
- Data leakage
- Infrastructure misconfiguration

Do not edit files. Provide findings with severity and concrete remediation.
```

#### 4. DevOps reviewer

```md
---
name: devops-reviewer
description: Reviews CI/CD, Kubernetes, Terraform, Helm, Docker, observability, and operational changes. Use for infrastructure or deployment changes.
tools: Read, Glob, Grep, Bash
model: sonnet
maxTurns: 25
memory: project
color: orange
---

You are a DevOps and cloud infrastructure reviewer.

Focus on:
- Deployment safety
- Rollback strategy
- Config drift
- Secret handling
- Observability
- Resource limits
- Environment differences
- Failure modes

Do not apply infrastructure changes. Recommend validation steps.
```

### Subagent hooks

Subagents can define hooks in frontmatter, and settings can define hooks that run when subagents start or stop. Anthropic documents two approaches: hooks in the subagent frontmatter, which run while that subagent is active, and `settings.json` hooks that run in the main session when subagents start or stop. Common subagent hook events are `PreToolUse`, `PostToolUse`, and `Stop`, which is converted to `SubagentStop` at runtime. ([Claude Code][8])

Example read-only database subagent:

```md
---
name: db-reader
description: Execute read-only database queries.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You may run only read-only SQL queries.
Never run INSERT, UPDATE, DELETE, DROP, CREATE, ALTER, or TRUNCATE.
```

Anthropic’s docs show this exact pattern: a `PreToolUse` hook validates Bash commands before execution and blocks SQL write operations. ([Claude Code][8])

---

## 9. Plugins: packaged extensions for teams

A **plugin** is a self-contained directory that can package multiple Claude Code components: skills, agents, hooks, MCP servers, LSP servers, monitors, output styles, and themes. Use plugins when you want to distribute a consistent set of capabilities across projects or teams, rather than copying individual skills and agents into every repo. ([Claude Code][5])

### Plugin structure

```text
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── code-review/
│       └── SKILL.md
├── agents/
│   └── security-reviewer.md
├── hooks/
│   └── hooks.json
├── scripts/
│   ├── security-scan.sh
│   └── format-code.sh
├── .mcp.json
├── .lsp.json
├── bin/
│   └── my-tool
├── LICENSE
└── CHANGELOG.md
```

Anthropic’s plugin reference says `.claude-plugin/plugin.json` contains metadata and configuration, while component directories like `skills/`, `agents/`, `hooks/`, `output-styles/`, `themes/`, and `monitors/` live at the plugin root, not inside `.claude-plugin`. It also states that a plugin-root `CLAUDE.md` is not loaded as project context; plugins contribute context through skills, agents, and hooks. ([Claude Code][5])

### Plugin manifest example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-plugin-manifest.json",
  "name": "team-engineering-tools",
  "displayName": "Team Engineering Tools",
  "version": "1.0.0",
  "description": "Shared Claude Code skills, agents, hooks, and MCP integrations for this engineering team.",
  "author": {
    "name": "Engineering Platform Team"
  },
  "skills": "./skills/",
  "agents": "./agents/",
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./.mcp.json",
  "keywords": ["engineering", "review", "testing", "security"]
}
```

If a plugin manifest exists, `name` is the only required field. The name is used for namespacing plugin components, such as `plugin-dev:agent-creator`. Claude Code ignores unrecognized top-level manifest fields, but type errors can still fail validation. ([Claude Code][5])

### Plugin hooks example

`hooks/hooks.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

Plugins can provide hooks from `hooks/hooks.json` or inline in `plugin.json`, and plugin hooks respond to the same lifecycle events as user-defined hooks, including `SessionStart`, `PreToolUse`, `PostToolUse`, `SubagentStart`, `SubagentStop`, and `Stop`. ([Claude Code][5])

### Creating and installing plugins

Claude Code supports plugin CLI commands such as `plugin init`, `plugin install`, `plugin uninstall`, `plugin enable`, `plugin disable`, `plugin update`, `plugin list`, and `plugin details`. `claude plugin init <name>` scaffolds a plugin under `~/.claude/skills/<name>/`, and `--with skills agents hooks mcp` can scaffold component folders. Installing with `--scope project` writes to `.claude/settings.json`, making the plugin available to everyone who clones the repository. ([Claude Code][5])

Examples:

```bash
claude plugin init team-engineering-tools --with skills agents hooks mcp
claude plugin install formatter@team-tools --scope project
claude plugin list
claude plugin details formatter@team-tools
claude plugin disable formatter@team-tools
```

---

## 10. Skill vs plugin vs subagent: when to use what

| Mechanism   | Best for                                 | Shape                               | Loaded when                   | Can run tools?              | Team shareable? |
| ----------- | ---------------------------------------- | ----------------------------------- | ----------------------------- | --------------------------- | --------------- |
| `CLAUDE.md` | Always-relevant repo context             | Markdown file                       | Session startup               | No direct execution         | Yes             |
| Rules       | Path-specific instructions               | Markdown files                      | When relevant path applies    | No direct execution         | Yes             |
| Skill       | Reusable task or reference               | `SKILL.md` folder                   | When invoked or auto-selected | Via allowed tools           | Yes             |
| Subagent    | Specialized worker with isolated context | Markdown file with frontmatter      | Spawned by Claude/user        | Yes, with tool limits       | Yes             |
| Hook        | Deterministic lifecycle automation       | JSON config + command/script/prompt | Lifecycle event               | Yes, outside model decision | Yes             |
| Plugin      | Packaged extension bundle                | Directory with manifest             | When enabled                  | Through bundled components  | Yes             |
| MCP server  | External tools/data access               | `.mcp.json`                         | When configured/approved      | Yes, exposes tools          | Yes             |

### Decision rules

Use **`CLAUDE.md`** when the instruction is always true and always useful: architecture overview, commands, conventions, safety rules, repo map.

Use a **skill** when the instruction is a repeatable workflow: review PR, write ADR, create release notes, debug failing tests, generate docs, prepare a deployment checklist.

Use a **subagent** when the task needs a focused worker or isolated context: security review, architecture review, test design, log analysis, database query validation, dependency review.

Use a **hook** when compliance must be deterministic: block dangerous commands, prevent reading secrets, run formatter after edits, require tests before final response, audit tool calls.

Use a **plugin** when you want to distribute a bundle: company-wide review workflow, shared security hooks, team-specific agents, MCP integrations, CLI helpers.

Use **MCP** when Claude needs new tools or external systems: GitHub, Jira, Slack, database, docs search, cloud APIs, internal services.

---

## 11. Practical setup for your kind of engineering repo

For a senior engineering/cloud/AI-agent repo, I would start with this minimal but strong setup:

```text
CLAUDE.md
.claude/settings.json
.claude/agents/repo-architect.md
.claude/agents/code-reviewer.md
.claude/agents/test-engineer.md
.claude/agents/security-reviewer.md
.claude/skills/implement-feature/SKILL.md
.claude/skills/review-pr/SKILL.md
.claude/skills/debug-failing-test/SKILL.md
.claude/skills/write-adr/SKILL.md
.claude/hooks/block-dangerous-commands.sh
.claude/hooks/run-format-after-edit.sh
```

Do not start by building a plugin unless you need reuse across many repositories. Start repo-local. Once the same `.claude/skills`, `.claude/agents`, and hooks are copied into two or three projects, promote them into a plugin.

---

## 12. Example complete `.claude/settings.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(git status *)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(npm test *)",
      "Bash(npm run build)",
      "Bash(make test)",
      "Bash(make lint)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./node_modules/**)",
      "Read(./dist/**)",
      "Read(./build/**)",
      "Bash(curl *)",
      "Bash(wget *)",
      "Bash(rm -rf /)",
      "Bash(rm -rf *)",
      "Bash(terraform apply *)",
      "Bash(terraform destroy *)",
      "Bash(kubectl delete *)",
      "Bash(helm uninstall *)",
      "Bash(npm publish *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-dangerous-commands.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/run-format-after-edit.sh"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "code-reviewer|security-reviewer|test-engineer",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Subagent completed. Review its findings before applying changes.'"
          }
        ]
      }
    ]
  }
}
```

This is intentionally conservative. The purpose is not to let Claude do everything automatically. The purpose is to give Claude enough safe autonomy for engineering productivity while keeping destructive commands, secrets, production actions, and broad infrastructure changes behind human judgment.

---

## 13. Example skill set for a professional repo

### `implement-feature`

```md
---
name: implement-feature
description: Implement a feature using the existing project architecture, with minimal diffs, relevant tests, and a final risk summary.
argument-hint: "[feature description]"
allowed-tools: Read, Grep, Glob, Bash, Edit, Write
---

## Goal

Implement the requested feature:

$ARGUMENTS

## Rules

- First inspect existing patterns.
- Prefer minimal changes.
- Do not introduce new dependencies unless necessary.
- Add or update tests.
- Do not change public APIs without explaining why.
- Do not commit.

## Workflow

1. Identify relevant files.
2. Summarize implementation plan.
3. Make the smallest useful change.
4. Run focused tests.
5. Report changed files, tests run, and risks.
```

### `debug-failing-test`

```md
---
name: debug-failing-test
description: Debug a failing test or error by finding the root cause, proposing a minimal fix, and validating with the narrowest relevant test.
argument-hint: "[test name or error]"
allowed-tools: Read, Grep, Glob, Bash, Edit
---

## Input

$ARGUMENTS

## Workflow

1. Reproduce or inspect the failure.
2. Identify the failing assertion or stack trace.
3. Trace the relevant code path.
4. Explain the likely root cause.
5. Apply the smallest fix only if requested or clearly safe.
6. Run the narrowest test.
7. Report confidence and remaining risks.
```

### `write-adr`

```md
---
name: write-adr
description: Create an Architecture Decision Record for a technical decision, including context, options, trade-offs, decision, and consequences.
argument-hint: "[decision topic]"
allowed-tools: Read, Grep, Glob, Write, Edit
---

Use `template.md` in this skill directory.

Create or update an ADR for:

$ARGUMENTS

Include:
1. Status
2. Context
3. Decision
4. Options considered
5. Trade-offs
6. Consequences
7. Rollback or revisit criteria
```

---

## 14. Example subagent set for a cloud/AI engineering repo

```text
.claude/agents/
├── repo-architect.md
├── backend-engineer.md
├── frontend-engineer.md
├── devops-reviewer.md
├── security-reviewer.md
├── test-engineer.md
├── docs-writer.md
└── release-manager.md
```

Do not create too many agents at the beginning. Subagents add routing complexity. Start with four: architect, reviewer, tester, security/devops. Add more only when repeated tasks become clearly separable.

---

## 15. Agent teams vs subagents

Subagents run within a single Claude Code session and report results back to the main agent. Agent teams are different: they coordinate multiple Claude Code instances, with one session acting as lead and teammates working independently with their own context windows. Agent teams are experimental, disabled by default, and require enabling `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. Anthropic says teams are best for parallel exploration, new modules, competing debugging hypotheses, and cross-layer coordination, but they add coordination overhead and cost more tokens. ([Claude Code][9])

Use **subagents** for focused worker tasks where only the final result matters.

Use **agent teams** when workers need to communicate, challenge each other, or coordinate independent streams of work.

For normal repo setup, start with subagents. Add agent teams only for larger design/research/debugging sessions.

---

## 16. Common anti-patterns

### Anti-pattern 1: giant `CLAUDE.md`

A huge `CLAUDE.md` increases token cost and reduces adherence. Anthropic recommends concise, specific instructions and a target under 200 lines. Move long procedures into skills and path-specific rules. ([Claude Code][2])

### Anti-pattern 2: using `CLAUDE.md` for enforcement

Do not rely on “Never run dangerous commands” in `CLAUDE.md` alone. Use `permissions.deny` and `PreToolUse` hooks.

### Anti-pattern 3: letting Claude auto-run side-effect skills

Deploy, publish, release, send message, delete, migrate, and production operations should use `disable-model-invocation: true`.

### Anti-pattern 4: too many subagents

If every task has an agent, routing becomes noisy. Start with a small set of high-value roles.

### Anti-pattern 5: putting plugin instructions in plugin `CLAUDE.md`

Anthropic states a plugin-root `CLAUDE.md` is not loaded as project context. Ship plugin instructions through skills, agents, or hooks instead. ([Claude Code][5])

### Anti-pattern 6: committing personal settings

Do not commit `CLAUDE.local.md` or `.claude/settings.local.json`.

---

## 17. Recommended rollout plan

### Phase 1: Basic repo guidance

Add:

```text
CLAUDE.md
.claude/settings.json
```

Focus on repo map, commands, coding standards, testing expectations, and secret protection.

### Phase 2: Repeatable workflows

Add skills:

```text
.claude/skills/review-pr/
.claude/skills/debug-failing-test/
.claude/skills/implement-feature/
.claude/skills/write-adr/
```

Use skills for workflows you would otherwise paste repeatedly into Claude.

### Phase 3: Specialized workers

Add subagents:

```text
.claude/agents/code-reviewer.md
.claude/agents/security-reviewer.md
.claude/agents/test-engineer.md
.claude/agents/repo-architect.md
```

Use them when you want context isolation or specialized review.

### Phase 4: Deterministic guardrails

Add hooks:

```text
.claude/hooks/block-dangerous-commands.sh
.claude/hooks/run-format-after-edit.sh
```

Use hooks only for checks that must happen automatically.

### Phase 5: Team distribution

When reused across multiple repos, convert to plugin:

```bash
claude plugin init team-engineering-tools --with skills agents hooks mcp
```

Use project-scope plugin installation when the team should share the same tooling. Claude’s plugin install command supports `--scope project`, which writes to `.claude/settings.json`. ([Claude Code][5])

---

## 18. Recommended default for your use case

Given your goal of using AI coding agents across project repos, tutorials, skills, and SaaS/product ideation, I would use this default:

```text
Start repo-local first:
- CLAUDE.md
- .claude/settings.json
- .claude/skills/
- .claude/agents/
- .claude/hooks/

Promote to plugin later:
- only after you reuse the same setup across multiple repos
```

For your own work, I would create one personal `~/.claude` setup for cross-project preferences and a project-level `.claude` setup per repo. For team or reusable “engineering operating system” workflows, package them as a plugin after the patterns stabilize.

[1]: https://code.claude.com/docs/en/claude-directory "Explore the .claude directory - Claude Code Docs"
[2]: https://code.claude.com/docs/en/memory "How Claude remembers your project - Claude Code Docs"
[3]: https://code.claude.com/docs/en/settings "Claude Code settings - Claude Code Docs"
[4]: https://code.claude.com/docs/en/hooks "Hooks reference - Claude Code Docs"
[5]: https://code.claude.com/docs/en/plugins-reference "Plugins reference - Claude Code Docs"
[6]: https://code.claude.com/docs/en/skills "Extend Claude with skills - Claude Code Docs"
[7]: https://docs.anthropic.com/en/docs/claude-code/sub-agents?utm_source=chatgpt.com "Create custom subagents - Claude Code Docs"
[8]: https://code.claude.com/docs/en/sub-agents "Create custom subagents - Claude Code Docs"
[9]: https://code.claude.com/docs/en/agent-teams "Orchestrate teams of Claude Code sessions - Claude Code Docs"
