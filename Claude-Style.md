# Claude Code: Practical Project Setup Guide

A research-backed, repo-oriented reference for setting up a **Claude Code-ready project repository**. Covers `CLAUDE.md`, `.claude/settings.json`, hooks, skills, subagents, plugins, MCP, and when to use each.

## TL;DR

- A Claude Code project is a normal source repo plus two opinionated additions: a `CLAUDE.md` memory file at the root and a `.claude/` directory containing `settings.json`, `agents/`, `skills/`, `commands/`, `hooks/`, and `.mcp.json`.
- **Skills, Subagents, and Plugins are not interchangeable**: Skills are progressively-disclosed prompt directories that load when relevant; Subagents are isolated Claude instances with their own context window; Plugins are packaging units that bundle skills, agents, hooks, and MCP servers for distribution.
- **Hooks are the only deterministic enforcement layer.** `CLAUDE.md` is context, not enforcement — a `PreToolUse` hook with exit code 2 is the only reliable hard block.

---

## 1. Mental model: what Claude Code reads from a repo

Claude Code is not configured by one file. A project can influence Claude through several layers:

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

The important distinction: **`CLAUDE.md` is context, not enforcement**. Claude reads it at startup and tries to follow it, but for deterministic behavior (blocking destructive commands, running formatters after edits), use settings, permissions, and hooks.

### 1.1 CLAUDE.md hierarchy (6 levels)

Claude Code supports a 6-level memory hierarchy. Files higher in the tree load in full at launch; subdirectory files load on demand.

| Level | Path | Scope | Loaded |
|---|---|---|---|
| Enterprise managed | (MDM/Group Policy path) | Org-wide policy | Always |
| Global user | `~/.claude/CLAUDE.md` | All your projects | Always |
| Project root | `<repo>/CLAUDE.md` | Whole repo | Always |
| Project local | `<repo>/CLAUDE.local.md` | Personal, gitignored | Always |
| Subdirectory | `<repo>/packages/api/CLAUDE.md` | Only that subtree | On-demand |
| Imported | Files referenced via `@path/to/file.md` | Modular sections | At launch |

`@import` syntax (e.g., `@.claude/rules/testing.md`) lets you split a large CLAUDE.md into modular files.

### 1.2 Global `~/.claude/` directory

```text
~/.claude/
├── CLAUDE.md                       # Global user-level memory
├── settings.json                   # Global user settings
├── agents/                         # Personal subagents (all projects)
├── skills/                         # Personal skills (all projects)
├── commands/                       # Personal slash commands
├── plugins/                        # CLI-installed plugins
├── projects/                       # Per-project session transcripts
└── .credentials.json               # OAuth tokens
~/.claude.json                      # Main state file: OAuth, MCP config, project trust
```

Note the two locations: `~/.claude/` is a directory; `~/.claude.json` is a separate top-level file. MCP user-scope and local-scope configs live in `~/.claude.json`; only the project-scope `.mcp.json` lives inside the project repo.

---

## 2. Recommended repo structure

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
    └── testing.md
```

**What to commit vs. gitignore:**
- Commit: `CLAUDE.md`, `.claude/settings.json`, `.claude/agents/`, `.claude/skills/`, `.claude/hooks/`, `.claude/rules/`, `.mcp.json`
- Gitignore: `CLAUDE.local.md`, `.claude/settings.local.json`, `.claude/logs/`, anything containing secrets

---

## 3. What should be in `CLAUDE.md`

Use `CLAUDE.md` for **always-relevant project context**. Target under 200 lines — longer files reduce adherence. Add content when Claude repeats the same mistake, when code review catches something Claude should have known, or when a new teammate would need the same context.

### Recommended `CLAUDE.md` template

````md
# Project Guide for Claude

## Project purpose

This repository contains [short description].
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

```bash
npm install
npm run lint
npm test
npm run build
```

Do not invent commands. If a command is missing, inspect `package.json`, `Makefile`, `README.md`, or ask.

## Coding standards

* Prefer small, focused changes.
* Preserve existing architecture unless explicitly asked to refactor.
* Do not change public APIs without calling it out.
* Avoid broad formatting-only diffs.

## Testing expectations

Before claiming a change is complete:
1. Run the smallest relevant test first.
2. Run lint/typecheck for typed code.
3. If tests cannot be run, explain why.

## Security and secrets

* Never read or print `.env`, `.env.*`, credentials, private keys.
* Never hardcode secrets.
* Treat migration scripts, infrastructure changes, and auth changes as high-risk.

## Git and PR behavior

* Do not commit unless explicitly asked.
* Keep diffs minimal.
* For PR descriptions, include: summary, tests, risks, rollback notes.
````

### What not to put in `CLAUDE.md`

Avoid large API references, long tutorials, full style guides, or multi-step playbooks. If an entry is a multi-step procedure or only matters for part of the codebase, move it to a skill or path-scoped rule.

---

## 4. `CLAUDE.local.md`: private personal context

Use `CLAUDE.local.md` for personal preferences that should not be committed. Add it to `.gitignore`.

```md
# Personal Claude Preferences

- Prefer minimal diffs unless I explicitly request refactoring.
- Explain architectural trade-offs before changing structure.
- When debugging tests, show the failing assertion and suspected root cause first.
- Do not run expensive integration tests unless I ask.
```

`.gitignore` entries:

```gitignore
CLAUDE.local.md
.claude/settings.local.json
```

---

## 5. `.claude/settings.json`: project configuration

`settings.json` is the official mechanism for configuring Claude Code through hierarchical settings.

### Config layer precedence (highest first)

1. Managed/Enterprise (MDM, `managed-settings.json`) — cannot be overridden
2. CLI flags (`--settings`, `--permission-mode`)
3. `.claude/settings.local.json` (project local, gitignored)
4. `.claude/settings.json` (project shared)
5. `~/.claude/settings.json` (user global)

Array settings (permission rules, hooks) are **concatenated and deduplicated** across layers, not replaced. Scalar values from higher layers override lower.

### Strong project baseline

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
      "Bash(kubectl delete *)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "0"
  }
}
```

### Permission rule syntax

`Tool` or `Tool(specifier)`:
- `Bash` — all Bash calls
- `Bash(npm run *)` — wildcard matching
- `Bash(* install)`, `Bash(git * main)` — wildcard anywhere
- `Read(src/**)`, `Read(*.md)` — gitignore-style globs
- `WebFetch(domain:docs.anthropic.com)` — domain-scoped
- `mcp__github__create-pull-request` — MCP tool (double underscore, no parens)
- `Task(Explore)` — restrict subagent types Claude may spawn

Evaluation order: **deny → ask → allow**. Deny always wins.

### Permission modes

- `default` — prompt on every privileged operation
- `acceptEdits` — auto-approve file writes; still confirm bash
- `plan` — Claude can analyze and propose but not execute
- `auto` — background classifier auto-approves safe ops
- `dontAsk` — anything not in `allow` is denied silently (good for headless agents)
- `bypassPermissions` — YOLO mode; only for sandboxed/CI environments

### Settings schema reference

The complete schema has 125+ keys. Key additional settings:

```json
{
  "model": "claude-sonnet-4-6",
  "autoUpdatesChannel": "stable",
  "permissions": {
    "defaultMode": "default",
    "ask": ["Write(**)", "Bash(git commit *)"],
    "additionalDirectories": ["~/shared/libraries"],
    "disableBypassPermissionsMode": "disable"
  },
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>"
  },
  "sandbox": {
    "allowedDomains": ["github.com", "registry.npmjs.org"]
  },
  "claudeMdExcludes": ["packages/legacy/**/CLAUDE.md"],
  "autoMemoryEnabled": true,
  "disableAllHooks": false
}
```

Schema URL `https://json.schemastore.org/claude-code-settings.json` provides editor autocomplete.

### API key and auth

- `ANTHROPIC_API_KEY` env var is the standard
- OAuth tokens live in `~/.claude/.credentials.json`
- Pro/Max subscriptions auth via the `/login` browser flow
- For Bedrock/Vertex: set `ANTHROPIC_BEDROCK_BASE_URL` / `ANTHROPIC_VERTEX_PROJECT_ID` plus standard cloud creds

---

## 6. Hooks: lifecycle automation and enforcement

Hooks are deterministic automation points — user-defined shell commands, HTTP endpoints, LLM prompts, or subagents that execute automatically at specific points in Claude Code's lifecycle. They run **outside the LLM** and are the only mechanism that guarantees behavior.

### 6.1 Complete lifecycle event list (30 events)

| Event | Fires When |
|---|---|
| `SessionStart` | A session begins or resumes |
| `Setup` | Started with `--init-only` or `--init`/`--maintenance` in `-p` mode |
| `UserPromptSubmit` | You submit a prompt, before Claude processes it |
| `UserPromptExpansion` | A user-typed command expands into a prompt |
| `PreToolUse` | Before a tool call executes — can block it |
| `PermissionRequest` | When a permission dialog appears |
| `PermissionDenied` | When a tool call is denied by the auto mode classifier |
| `PostToolUse` | After a tool call succeeds |
| `PostToolUseFailure` | After a tool call fails |
| `PostToolBatch` | After a batch of parallel tool calls resolves |
| `Notification` | When Claude Code sends a notification |
| `MessageDisplay` | While assistant message text is displayed |
| `SubagentStart` | When a subagent is spawned |
| `SubagentStop` | When a subagent finishes |
| `TaskCreated` | When a task is created via `TaskCreate` |
| `TaskCompleted` | When a task is marked as completed |
| `Stop` | When Claude finishes responding |
| `StopFailure` | When the turn ends due to an API error |
| `TeammateIdle` | When an agent team teammate is about to go idle |
| `InstructionsLoaded` | When a `CLAUDE.md` or `.claude/rules/*.md` is loaded |
| `ConfigChange` | When a configuration file changes during a session |
| `CwdChanged` | When the working directory changes (`cd`) — useful with direnv |
| `FileChanged` | When a watched file changes on disk (matcher is the filename glob) |
| `WorktreeCreate` | When a worktree is created |
| `WorktreeRemove` | When a worktree is removed |
| `PreCompact` | Before context compaction |
| `PostCompact` | After context compaction completes |
| `Elicitation` | When an MCP server requests user input |
| `ElicitationResult` | After a user responds to MCP elicitation |
| `SessionEnd` | When a session terminates |

### 6.2 Handler types (4 supported)

1. **`command`** — shell command, receives JSON on stdin, returns via exit code + stdout/stderr
2. **`http`** — POSTs the event JSON to a URL, expects JSON response (shipped v2.1.66, March 2026). `headers` supports `${VAR}` interpolation for vars listed in `allowedEnvVars`
3. **`prompt`** — single-turn LLM evaluation; sends prompt to a model for an `allow`/`deny`/`additionalContext` decision
4. **`agent`** — spawns a subagent with tool access for multi-step verification

All handler types support `timeout` (seconds), `async: true` (non-blocking, fire-and-forget), and the path placeholders `$CLAUDE_PROJECT_DIR`, `$CLAUDE_PLUGIN_ROOT`, `$CLAUDE_PLUGIN_DATA`.

### 6.3 Hook input/output contract

**Stdin (command) / POST body (http):**
```json
{
  "hook_event_name": "PreToolUse",
  "session_id": "...",
  "cwd": "/your/project",
  "transcript_path": "/tmp/transcript.jsonl",
  "tool_name": "Bash",
  "tool_input": { "command": "rm -rf dist/" },
  "tool_use_id": "..."
}
```

**Exit codes:**
- `0` — allow / no action
- `1` — non-blocking error (logged, execution continues)
- `2` — **blocking** for PreToolUse; *forces continuation* for Stop hooks

**Structured JSON output (more precise than exit codes):**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Edits to dist/ are blocked",
    "updatedInput": { "command": "modified-command" },
    "additionalContext": "Extra info Claude will see"
  }
}
```

`permissionDecision` accepts `"allow"`, `"deny"`, or `"ask"`. `updatedInput` lets a PreToolUse hook transparently modify tool arguments before execution — invisible to Claude.

### 6.4 Environment variables available to hooks

- `CLAUDE_PROJECT_DIR` — absolute path to the project root
- `CLAUDE_PLUGIN_ROOT`, `CLAUDE_PLUGIN_DATA` — when the hook is from a plugin
- `CLAUDE_SESSION_ID` (v2.1.9+)
- `CLAUDE_ENV_FILE` — in `SessionStart`/`CwdChanged`/`FileChanged` hooks; write `export VAR=value` lines here to persist env vars for the session
- `CLAUDE_TOOL_INPUT_FILE_PATH` — set on PostToolUse for file-modifying tools

**Windows caveat:** On VS Code for Windows (GitHub issue #16564), `TOOL_NAME` and `EXIT_CODE` env vars may not be populated. Parse stdin JSON instead of relying on those env vars.

### 6.5 Configuration schema

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/guard.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          { "type": "command", "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\"" }
        ]
      }
    ]
  }
}
```

Hooks from managed > project > user > local > plugin settings are **merged**, not overwritten — every matching hook from every layer runs.

### 6.6 Practical hook examples

**Block dangerous commands (PreToolUse):**

`.claude/hooks/block-dangerous-commands.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT="$(cat)"
COMMAND="$(echo "$INPUT" | jq -r '.tool_input.command // empty')"

if echo "$COMMAND" | grep -Eiq 'rm -rf /|rm -rf \*|kubectl delete|terraform apply|terraform destroy|npm publish|docker system prune'; then
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

**Run formatter after edits (PostToolUse):**

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

**Block Stop until tests pass:**

```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then exit 0; fi
if ! npm test --silent 2>/dev/null; then
  echo "Tests are failing. Fix them before finishing." >&2
  exit 2
fi
exit 0
```

**Inject branch context on session start:**

```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "echo '{\"additionalContext\": \"Branch: '$(git branch --show-current)'\"}'"
      }]
    }]
  }
}
```

### 6.7 Hooks in skill/subagent frontmatter

Skills and subagents can declare scoped hooks in frontmatter (only `PreToolUse`, `PostToolUse`, `Stop`). The `once: true` field fires the hook only once per session:

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check.sh"
          once: true
---
```

---

## 7. Skills: reusable procedures and reference knowledge

A **skill** is a reusable instruction package. Skills are directories with a `SKILL.md` entry point and optional supporting files. Claude auto-invokes skills when relevant, or users invoke them with `/skill-name`.

**Progressive disclosure**: at session start Claude reads only skill frontmatter (name + description, ~100 tokens per skill). The full body loads only when the skill is invoked. Helper files in `scripts/`, `references/`, `assets/` load only when the skill body references them. This makes hundreds of skills practical without context bloat.

### Skill locations

| Scope | Path |
|---|---|
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` |
| Project | `.claude/skills/<skill-name>/SKILL.md` |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` |
| Enterprise | managed settings |

### Complete SKILL.md frontmatter fields

| Field | Type | Description |
|---|---|---|
| `name` | string (defaults to directory name) | Display name in skill listings |
| `description` | string | What the skill does and when to use it — primary trigger signal |
| `when_to_use` | string | Extra trigger phrases; combined cap of 1,536 chars with description |
| `argument-hint` | string | Autocomplete hint, e.g. `[issue-number]` |
| `arguments` | string or list | Named positional args for `$name` substitution |
| `disable-model-invocation` | boolean (default `false`) | Hide from auto-invocation; only `/name` triggers |
| `user-invocable` | boolean (default `true`) | Set `false` to hide from `/` menu |
| `allowed-tools` | list or string | Tools auto-approved while this skill is active |
| `disallowed-tools` | list or string | Tools removed while this skill is active |
| `model` | model alias / full ID / `inherit` | Override model for this skill's turn |
| `effort` | `low`/`medium`/`high`/`xhigh`/`max` | Override session effort |
| `context` | `fork` | Run in a forked subagent context |
| `agent` | subagent type name | Which subagent type when `context: fork` |
| `hooks` | object | Scoped `PreToolUse`/`PostToolUse`/`Stop` hooks |
| `paths` | list of globs | Only auto-activate when working on matching files |
| `shell` | `bash` (default) or `powershell` | Shell for inline `` !`cmd` `` blocks |

Note: `metadata:` frontmatter is not in Anthropic's spec — community templates include it, but Claude Code does not consume those keys.

### Dynamic context injection

The `` !`command` `` syntax runs a shell command and inlines its output before Claude sees the skill content:

```markdown
---
name: summarize-changes
description: Summarizes uncommitted changes and flags risks. Use when the user asks what changed, wants a commit message, or asks to review their diff.
allowed-tools: Bash(git *), Read
---

## Current changes
!`git diff HEAD`

## Instructions
Summarize the changes above in two or three bullet points, then list any risks.
```

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

Review the current changes against `$ARGUMENTS`. If no base branch is provided, use `main`.

## Steps

1. Inspect the current branch and changed files.
2. Read only files relevant to the change.
3. Identify correctness risks, security risks, maintainability issues, and missing tests.
4. Suggest minimal changes first.
5. Do not modify files unless the user explicitly asks.

## Commands

```bash
git diff $ARGUMENTS...HEAD
git status --short
```

## Output format

1. Summary
2. High-risk issues
3. Medium-risk issues
4. Test gaps
5. Suggested next actions
````

### When to disable automatic skill invocation

Use `disable-model-invocation: true` for workflows with side effects (deployment, publishing, sending messages). Claude should not decide on its own that a deployment should happen.

```md
---
name: deploy-production
description: Deploy production after explicit user approval.
disable-model-invocation: true
allowed-tools: Bash
---

Only run when the user explicitly invokes `/deploy-production`.

Before deploying:
1. Confirm branch.
2. Confirm clean working tree.
3. Confirm tests passed.
4. Ask for explicit final approval.
```

### Supporting files inside skills

```text
.claude/skills/create-architecture-doc/
├── SKILL.md
├── template.md
├── examples/
│   └── architecture-decision-record.md
└── scripts/
    └── validate-links.sh
```

Keep `SKILL.md` under 500 lines and reference supporting files for details.

**Note:** Custom commands have been merged into skills. `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both produce `/deploy`. Skills are recommended for new work.

---

## 8. Subagents: specialized workers with isolated context

A **subagent** is a separate Claude instance with its own system prompt, its own 200K-token context window, its own tool allowlist, and its own permission mode. The main agent delegates a task; the subagent works in isolation and returns only its final output. Intermediate file reads, search results, and exploration stay in the subagent's context — they never pollute the main conversation.

**The point is context preservation, not capability.** When sessions exceed ~70% context, response quality degrades. Isolating exploration in subagents keeps the main thread sharp.

**Built-in subagents:** `Explore` (read-only, defaults to Haiku), `Plan` (context + strategy), `General-purpose` (exploration + modification).

### Subagent locations

| Scope | Path |
|---|---|
| Managed | `.claude/agents/` in managed settings dir |
| Project | `.claude/agents/` (committed to repo) |
| Personal | `~/.claude/agents/` (all projects) |
| Plugin | `<plugin>/agents/` (namespaced) |

### Complete subagent frontmatter fields

| Field | Type / Default | Description |
|---|---|---|
| `name` | lowercase + hyphens (required) | Unique ID; passed to hooks as `agent_type` |
| `description` | string (required) | When to delegate to this subagent |
| `tools` | comma-separated; inherits all if omitted | Tool allowlist |
| `disallowedTools` | comma-separated | Tools to subtract |
| `model` | `sonnet`/`opus`/`haiku`/full ID/`inherit` | Model choice (default `inherit`) |
| `permissionMode` | `default`/`acceptEdits`/`auto`/`dontAsk`/`bypassPermissions`/`plan` | (Ignored for plugin subagents) |
| `maxTurns` | number | Agentic-turn cap |
| `skills` | list | Preload skills into context at startup |
| `mcpServers` | list of names or inline configs | MCP servers available |
| `hooks` | object | Lifecycle hooks |
| `memory` | `user`/`project`/`local` | Persistent memory scope |
| `background` | boolean (default `false`) | Always run as background task |
| `effort` | `low`/`medium`/`high`/`xhigh`/`max` | Override session effort |
| `isolation` | `worktree` | Run in a temporary git worktree |
| `color` | `red`/`blue`/`green`/`yellow`/`purple`/`orange`/`pink`/`cyan` | Display color |
| `initialPrompt` | string | Auto-submitted first user turn |

**Cost note:** Subagents are billed on the same plan as the main session. Agent teams use ~7x more tokens than standard sessions when teammates run in plan mode.

### Subagent templates

**Code reviewer:**

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

Focus on: correctness, security, maintainability, test coverage, backward compatibility, and consistency with existing patterns. Do not edit files unless explicitly asked.

Review process:
1. Inspect changed files.
2. Read relevant surrounding code.
3. Identify high-risk issues first.
4. Suggest minimal fixes.
5. State what tests should be run.

Output: Summary → Critical issues → Suggested improvements → Missing tests → Confidence level
```

**Repo architect:**

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

Analyze: current architecture, module boundaries, dependency direction, scalability concerns, maintainability risks, operational risks.

Do not edit files. Return options, trade-offs, and a recommended default.
```

**Test engineer:**

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

Your job: identify missing test cases, add minimal tests using existing project patterns, prefer focused unit tests before broad integration tests, avoid rewriting unrelated tests.
```

**Security reviewer:**

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

Focus on: secrets exposure, injection risks, auth and authorization logic, unsafe shell commands, dependency risk, data leakage, infrastructure misconfiguration.

Do not edit files. Provide findings with severity and concrete remediation.
```

**DevOps reviewer:**

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

Focus on: deployment safety, rollback strategy, config drift, secret handling, observability, resource limits, environment differences, failure modes.

Do not apply infrastructure changes. Recommend validation steps.
```

### Subagent hooks

Subagents can define hooks in frontmatter that run while that subagent is active:

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

---

## 9. Plugins: packaged extensions for teams

A **plugin** is a self-contained directory that packages multiple Claude Code components: skills, agents, hooks, MCP servers, LSP servers, monitors, output styles, and themes. Use plugins when you want to distribute a consistent set of capabilities across projects or teams.

### Plugin structure

```text
my-plugin/
├── .claude-plugin/
│   └── plugin.json           # only this file belongs here
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
└── README.md
```

**Critical layout rule:** only `plugin.json` belongs inside `.claude-plugin/`; all components live at the plugin root. A plugin-root `CLAUDE.md` is NOT loaded as project context — contribute context through skills, agents, and hooks.

**Namespacing:** Plugin skills are exposed as `/plugin-name:skill-name`.

### Plugin manifest example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-plugin-manifest.json",
  "name": "team-engineering-tools",
  "displayName": "Team Engineering Tools",
  "version": "1.0.0",
  "description": "Shared skills, agents, hooks, and MCP integrations.",
  "author": { "name": "Engineering Platform Team" },
  "userConfig": {
    "apiKey": { "description": "API token", "sensitive": true }
  }
}
```

Only `name` is required. `userConfig` values declared `sensitive: true` are stored in the OS keychain.

### Plugin hooks

`hooks/hooks.json`:

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

### Creating and installing plugins

```bash
claude plugin init team-engineering-tools --with skills agents hooks mcp
claude plugin install formatter@team-tools --scope project
claude plugin list
claude plugin details formatter@team-tools
claude plugin disable formatter@team-tools
```

`--scope project` writes to `.claude/settings.json`, making the plugin available to everyone who clones the repo. Run `/reload-plugins` to pick up changes without restarting.

---

## 10. MCP (Model Context Protocol) Integration

MCP servers expose external tools and data to Claude — GitHub, Jira, Slack, databases, cloud APIs, internal services.

### Three scopes

| Scope | File | Shared? |
|---|---|---|
| Local (default) | `~/.claude.json` (per-project section) | Private to you |
| **Project** | `.mcp.json` at repo root | **Committed, team-shared** |
| User | `~/.claude.json` (top-level `mcpServers`) | All your projects |

Precedence: local > project > user.

### `.mcp.json` format

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": { "Authorization": "Bearer ${GITHUB_PAT}" }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"],
      "timeout": 600000
    },
    "memory": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

`${VAR}` syntax pulls from your environment so secrets stay out of git. `streamable-http` is accepted as an alias for `http`. SSE is deprecated.

### CLI management

```bash
# Add an HTTP server at project scope
claude mcp add --transport http sentry --scope project https://mcp.sentry.dev/mcp

# Add a stdio server at user scope
claude mcp add --scope user notes -- npx @modelcontextprotocol/server-notes ~/notes

# List, get, remove
claude mcp list
claude mcp get github
claude mcp remove github
```

For OAuth-authenticated servers, run `/mcp` inside Claude Code to trigger the browser flow.

The first time a project's `.mcp.json` is loaded, Claude Code asks for approval. Choose "Use this and all future MCP servers in this project" once to skip future prompts.

---

## 11. Security: defense in depth

The recommended security posture stacks four layers:

1. **Permission deny rules** (`settings.json`) — Claude won't attempt restricted operations
2. **Sandboxing** (`/sandbox`, Seatbelt on macOS, bubblewrap on Linux) — OS-level enforcement on Bash and its children
3. **PreToolUse hooks** — programmatic last line of defense; exit 2 hard-blocks
4. **OS file permissions** — for things that matter, don't rely on the AI layer

**Known gap:** `.env` deny rules are not consistently enforced — a Python or Node script invoked from Bash can read whatever the user can read. Sensitive secrets require hooks + sandboxing + OS permissions, not deny rules alone.

**Recommended baseline `.claude/settings.json`:**

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Bash(npm run *)", "Bash(npm test*)", "Bash(npx prettier --write *)",
      "Bash(git status)", "Bash(git diff *)", "Bash(git log *)",
      "Bash(git add *)", "Bash(git commit *)",
      "Read(**)", "Glob(**)", "Grep(**)"
    ],
    "ask": ["Write(**)", "Edit(**)", "Bash(git push *)"],
    "deny": [
      "Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)", "Read(~/.ssh/**)",
      "Bash(rm -rf *)", "Bash(curl * | bash)", "Bash(wget * | bash)",
      "Bash(git push --force *)", "Bash(git reset --hard *)",
      "Edit(.github/workflows/**)", "Edit(.claude/settings.json)",
      "Edit(Dockerfile*)", "Edit(terraform/**)"
    ]
  },
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write|Edit|MultiEdit",
      "hooks": [{ "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/protect-files.sh" }]
    }],
    "PostToolUse": [{
      "matcher": "Write|Edit|MultiEdit",
      "hooks": [{ "type": "command", "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\"" }]
    }]
  }
}
```

**File protection hook:**

`.claude/hooks/protect-files.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
PROTECTED=(".env" ".env.local" "secrets/" ".git/" "package-lock.json" "pnpm-lock.yaml")
for pattern in "${PROTECTED[@]}"; do
  if [[ "$FILE" == *"$pattern"* ]]; then
    echo "Protected file: $pattern" >&2
    exit 2
  fi
done
exit 0
```

---

## 12. Skill vs plugin vs subagent: when to use what

| Mechanism | Best for | Loaded when | Can run tools? | Team shareable? |
|---|---|---|---|---|
| `CLAUDE.md` | Always-relevant repo context | Session startup | No direct execution | Yes |
| Rules | Path-specific instructions | When relevant path applies | No direct execution | Yes |
| Skill | Reusable task or reference | When invoked or auto-selected | Via allowed tools | Yes |
| Subagent | Specialized worker, isolated context | Spawned by Claude/user | Yes, with tool limits | Yes |
| Hook | Deterministic lifecycle automation | Lifecycle event | Yes, outside model | Yes |
| Plugin | Packaged extension bundle | When enabled | Through bundled components | Yes |
| MCP server | External tools/data access | When configured/approved | Yes, exposes tools | Yes |

### Decision rules

- **`CLAUDE.md`** — always true and always useful: architecture, commands, conventions, safety rules
- **Skill** — repeatable workflow: review PR, write ADR, debug tests, generate release notes
- **Subagent** — focused worker or isolated context: security review, architecture analysis, log analysis
- **Hook** — must be deterministic: block commands, prevent secret access, run formatter, require tests
- **Plugin** — distribute a bundle: company-wide review workflow, shared hooks, team MCP integrations
- **MCP** — Claude needs new tools: GitHub, Jira, Slack, database, cloud APIs

---

## 13. Complete `.claude/settings.json` example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(git status *)", "Bash(git diff *)", "Bash(git log *)",
      "Bash(npm run lint)", "Bash(npm run test *)", "Bash(npm test *)",
      "Bash(npm run build)", "Bash(make test)", "Bash(make lint)"
    ],
    "deny": [
      "Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)",
      "Read(./config/credentials.json)", "Read(./node_modules/**)",
      "Bash(curl *)", "Bash(wget *)", "Bash(rm -rf /)", "Bash(rm -rf *)",
      "Bash(terraform apply *)", "Bash(terraform destroy *)",
      "Bash(kubectl delete *)", "Bash(helm uninstall *)", "Bash(npm publish *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/block-dangerous-commands.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/run-format-after-edit.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "code-reviewer|security-reviewer|test-engineer",
        "hooks": [
          { "type": "command", "command": "echo 'Subagent completed. Review its findings before applying changes.'" }
        ]
      }
    ]
  }
}
```

---

## 14. Example skill set

**`implement-feature`:**

```md
---
name: implement-feature
description: Implement a feature using the existing project architecture, with minimal diffs, relevant tests, and a final risk summary.
argument-hint: "[feature description]"
allowed-tools: Read, Grep, Glob, Bash, Edit, Write
---

## Goal
Implement: $ARGUMENTS

## Rules
- First inspect existing patterns.
- Prefer minimal changes.
- Do not introduce new dependencies unless necessary.
- Add or update tests.
- Do not commit.

## Workflow
1. Identify relevant files.
2. Summarize implementation plan.
3. Make the smallest useful change.
4. Run focused tests.
5. Report changed files, tests run, and risks.
```

**`debug-failing-test`:**

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

**`write-adr`:**

```md
---
name: write-adr
description: Create an Architecture Decision Record for a technical decision, including context, options, trade-offs, decision, and consequences.
argument-hint: "[decision topic]"
allowed-tools: Read, Grep, Glob, Write, Edit
---

Use `template.md` in this skill directory.

Create or update an ADR for: $ARGUMENTS

Include: Status, Context, Decision, Options considered, Trade-offs, Consequences, Rollback criteria.
```

---

## 15. Agent teams vs subagents

Subagents run within a single Claude Code session and report results back to the main agent.

Agent teams coordinate multiple Claude Code instances — one session acts as lead and teammates work independently with their own context windows. Agent teams are experimental (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), require v2.1.32+, and add coordination overhead plus ~7x token cost in plan mode.

Use **subagents** for focused worker tasks where only the final result matters.

Use **agent teams** when workers need to communicate, challenge each other, or coordinate independent streams of work.

For normal repo setup, start with subagents. Add agent teams only for larger design or debugging sessions.

---

## 16. Common anti-patterns

**Giant `CLAUDE.md`:** A huge file increases token cost and reduces adherence. Target under 200 lines. Move procedures into skills and path-specific rules into `.claude/rules/`.

**Using `CLAUDE.md` for enforcement:** Do not rely on "Never run dangerous commands" in `CLAUDE.md` alone. Use `permissions.deny` and `PreToolUse` hooks.

**Letting Claude auto-run side-effect skills:** Deploy, publish, release, send message, delete, and production operations must use `disable-model-invocation: true`.

**Too many subagents:** If every task has an agent, routing becomes noisy. Start with a small set of high-value roles (architect, reviewer, tester, security).

**Plugin instructions in plugin `CLAUDE.md`:** A plugin-root `CLAUDE.md` is not loaded as project context. Ship plugin instructions through skills, agents, or hooks.

**Committing personal settings:** Do not commit `CLAUDE.local.md` or `.claude/settings.local.json`.

**Over-trusting deny rules for secrets:** Deny rules protect Claude Code's file tools but not arbitrary scripts invoked via Bash. Combine deny rules + sandboxing + hooks + OS permissions.

---

## 17. Recommended rollout plan

### Phase 1: Basic repo guidance

```text
CLAUDE.md
.claude/settings.json
```

Focus: repo map, commands, coding standards, testing expectations, secret protection.

### Phase 2: Repeatable workflows

```text
.claude/skills/review-pr/
.claude/skills/debug-failing-test/
.claude/skills/implement-feature/
.claude/skills/write-adr/
```

Use skills for workflows you would otherwise paste repeatedly into Claude.

### Phase 3: Specialized workers

```text
.claude/agents/code-reviewer.md
.claude/agents/security-reviewer.md
.claude/agents/test-engineer.md
.claude/agents/repo-architect.md
```

Use subagents when you want context isolation or specialized review.

### Phase 4: Deterministic guardrails

```text
.claude/hooks/block-dangerous-commands.sh
.claude/hooks/run-format-after-edit.sh
```

Use hooks only for checks that must happen automatically.

### Phase 5: Team distribution

```bash
claude plugin init team-engineering-tools --with skills agents hooks mcp
claude plugin install team-engineering-tools --scope project
```

Promote to plugin only after the same setup is reused across multiple repos.

---

## 18. Thresholds and caveats

### When to revisit your setup

- `CLAUDE.md` exceeds 200 lines or 2,000 tokens → split into `@`-imported rule files
- A session regularly exceeds 70% context → lean harder on subagents for exploration
- Hooks add > 200 ms per tool call → switch slow ones to `async: true` or move to `Stop`
- You manually approve the same MCP tool 5+ times → add it to `permissions.allow`

### Caveats

1. **Documentation churn.** Claude Code is on a fast release cadence. Async hooks shipped January 2026, HTTP hooks in v2.1.66 March 2026, Agent Teams in v2.1.32 February 2026. Always check `claude --version` and `https://code.claude.com/docs/en/` for version-specific reference.

2. **Skills under-trigger.** Anthropic acknowledges this — descriptions should be deliberately keyword-rich and "pushy": "Use this skill when the user mentions X, Y, or Z, even if they don't say the exact term."

3. **`.env` deny rules can leak.** Multiple GitHub issues show Claude reading `.env` despite explicit deny rules when Bash subprocesses are used. PreToolUse hooks + sandboxing are the only reliable protection.

4. **Windows hook env vars.** Per GitHub issue #16564, VS Code on Windows does not populate `TOOL_NAME` and `EXIT_CODE` env vars. Parse stdin JSON instead.

5. **Two "plugins" exist.** Claude *Code* plugins (covered here) are dev-tool packages installed via `claude plugin install`. Claude *Cowork* plugins are knowledge-worker bundles installed from `claude.com/plugins` — same file format, different runtime. Hooks and subagents inside a Cowork plugin only run in the Cowork tab.

6. **`CLAUDE.local.md` is deprecated** in favor of `~/.claude/CLAUDE.md` + `.claude/settings.local.json` — old guides still reference it; new projects should not adopt it.

[1]: https://code.claude.com/docs/en/claude-directory "Explore the .claude directory"
[2]: https://code.claude.com/docs/en/memory "How Claude remembers your project"
[3]: https://code.claude.com/docs/en/settings "Claude Code settings"
[4]: https://code.claude.com/docs/en/hooks "Hooks reference"
[5]: https://code.claude.com/docs/en/plugins-reference "Plugins reference"
[6]: https://code.claude.com/docs/en/skills "Extend Claude with skills"
[7]: https://code.claude.com/docs/en/sub-agents "Create custom subagents"
[8]: https://code.claude.com/docs/en/agent-teams "Orchestrate agent teams"
