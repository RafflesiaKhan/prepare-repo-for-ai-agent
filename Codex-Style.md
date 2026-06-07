Below is a research-backed report for setting up a **Codex-ready project repository**, with the current Codex conventions as of June 2026. The most important correction is this: **Codex does not primarily use `CODEX.md` as the repo instruction file. It uses `AGENTS.md`.** You can configure fallback names, but `AGENTS.md` is the standard Codex reads automatically. ([OpenAI Developers][1])

# Codex project repo setup: detailed report

## 1. Core mental model

Codex repo customization has several layers:

```text
your-repo/
├── AGENTS.md
├── AGENTS.override.md                 # optional, stronger local override
├── .codex/
│   ├── config.toml                    # project-scoped Codex config
│   ├── hooks.json                     # lifecycle hooks
│   ├── rules/
│   │   └── default.rules              # command approval/blocking rules
│   ├── agents/
│   │   ├── pr-explorer.toml
│   │   ├── reviewer.toml
│   │   └── security-reviewer.toml
│   └── hooks/
│       ├── pre_tool_use_policy.py
│       ├── post_tool_use_review.py
│       └── stop_check.py
├── .agents/
│   ├── skills/
│   │   ├── review-pr/
│   │   │   └── SKILL.md
│   │   ├── implement-feature/
│   │   │   └── SKILL.md
│   │   └── write-adr/
│   │       └── SKILL.md
│   └── plugins/
│       └── marketplace.json
├── plugins/
│   └── team-engineering-plugin/
│       ├── .codex-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── review-pr/
│               └── SKILL.md
└── README.md
```

Codex uses:

| Layer                                      | Purpose                                     |
| ------------------------------------------ | ------------------------------------------- |
| `AGENTS.md`                                | Repository instructions and project context |
| `.codex/config.toml`                       | Project-scoped configuration                |
| `.codex/hooks.json` or `[hooks]` in config | Lifecycle automation                        |
| `.codex/rules/*.rules`                     | Command approval/blocking rules             |
| `.agents/skills/*/SKILL.md`                | Reusable task workflows                     |
| `.codex/agents/*.toml`                     | Custom subagent definitions                 |
| `.agents/plugins/marketplace.json`         | Repo-local plugin catalog                   |
| `plugins/*/.codex-plugin/plugin.json`      | Installable plugin packages                 |
| MCP config in `config.toml`                | External tools and data access              |

Codex also has personal/global layers, especially:

```text
~/.codex/config.toml
~/.codex/AGENTS.md
~/.codex/hooks.json
~/.codex/rules/
~/.codex/agents/
~/.agents/skills/
~/.agents/plugins/marketplace.json
```

Project-local `.codex/` configuration is only loaded when the project is trusted. If the project is untrusted, Codex skips project config, hooks, and rules, while still loading user and system config. ([OpenAI Developers][2])

---

# 2. `AGENTS.md`: Codex’s equivalent of project instructions

## 2.1 Does Codex use `CODEX.md`?

Based on current OpenAI Codex documentation, the standard instruction file is **`AGENTS.md`**, not `CODEX.md`. Codex reads `AGENTS.md` files before doing work and layers global guidance with project-specific overrides. ([OpenAI Developers][1])

You can configure fallback filenames with `project_doc_fallback_filenames`, for example:

```toml
project_doc_fallback_filenames = ["CODEX.md", "TEAM_GUIDE.md", ".agents.md"]
project_doc_max_bytes = 65536
```

But unless you configure that, `CODEX.md` is not the default project instruction file. Codex checks each directory in this order: `AGENTS.override.md`, then `AGENTS.md`, then fallback names configured in `project_doc_fallback_filenames`. ([OpenAI Developers][1])

## 2.2 How Codex discovers `AGENTS.md`

Codex builds an instruction chain at startup. It first reads global instructions from `~/.codex/AGENTS.override.md` or `~/.codex/AGENTS.md`. Then it walks from the project root down to the current working directory, reading at most one instruction file per directory. Files closer to the current directory appear later in the combined prompt, so they override broader guidance. Codex stops once the combined instruction size reaches `project_doc_max_bytes`, which defaults to 32 KiB. ([OpenAI Developers][1])

Example:

```text
repo/
├── AGENTS.md
└── services/
    ├── AGENTS.md
    └── payments/
        ├── AGENTS.override.md
        └── src/
```

If you start Codex inside `services/payments`, the effective order is:

```text
~/.codex/AGENTS.md
repo/AGENTS.md
repo/services/AGENTS.md
repo/services/payments/AGENTS.override.md
```

If `AGENTS.override.md` exists in a directory, Codex ignores that directory’s `AGENTS.md`. ([OpenAI Developers][1])

---

# 3. What should be in `AGENTS.md`

Use `AGENTS.md` for **stable, always-relevant repo guidance**.

Good content:

````md
# AGENTS.md

## Project purpose

This repository contains [short explanation of product/system].

The main user outcome is [business/user value].

## Tech stack

- Frontend: React, TypeScript, Carbon Design System
- Backend: Node.js / Python / Go
- Database: PostgreSQL / DynamoDB
- Infrastructure: Terraform, Kubernetes, Helm, Argo CD
- Tests: Jest, Pytest, Go test

## Repository map

- `src/`: application source code
- `src/components/`: reusable UI components
- `src/services/`: API and business logic
- `src/hooks/`: React hooks
- `tests/`: unit and integration tests
- `infra/`: infrastructure-as-code
- `docs/`: architecture and operational documentation

## Local development

Use existing package manager and scripts.

Before adding new commands, inspect:
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

* Prefer minimal, focused diffs.
* Follow existing architecture and naming.
* Do not introduce new dependencies without explaining why.
* Preserve public APIs unless the task explicitly requires a breaking change.
* Avoid broad formatting-only changes.
* Do not delete tests unless they are obsolete and the reason is documented.

## Testing expectations

Before claiming a change is complete:

1. Run the smallest relevant test first.
2. Run lint/typecheck when touching typed code.
3. Run the broader test suite only for cross-cutting changes.
4. If tests cannot run, explain the exact reason and what should be run manually.

## Security and secrets

* Never read, print, or modify `.env`, `.env.*`, private keys, credentials, tokens, or production secret files.
* Never hardcode secrets.
* Treat auth, payment, infrastructure, migration, and permission changes as high-risk.
* Do not run destructive infrastructure commands unless explicitly requested.

## Git behavior

* Do not commit unless explicitly asked.
* Do not push unless explicitly asked.
* Summarize changed files, tests run, and residual risks.
* For PR summaries, include: summary, tests, risks, rollback notes.

## Communication

* State assumptions.
* Explain trade-offs for architectural changes.
* Prefer practical implementation guidance over generic advice.

````

Do not put long API references, huge architecture documents, complete tutorials, or large checklists in `AGENTS.md`. Put those in skills, docs, or module-specific `AGENTS.md` files.

---

# 4. `.codex/config.toml`: project and user configuration

Codex uses `config.toml`, not JSON, for main configuration. Personal defaults live in `~/.codex/config.toml`; project overrides go in `.codex/config.toml`; CLI flags and `--config` overrides have the highest precedence. Project config is loaded only for trusted projects. :contentReference[oaicite:6]{index=6}

## 4.1 Configuration precedence

Highest to lowest:

1. CLI flags and `--config` overrides
2. Project config files, from root to current directory, closest wins
3. Profile files selected with `--profile`
4. User config at `~/.codex/config.toml`
5. System config, such as `/etc/codex/config.toml`
6. Built-in defaults

:contentReference[oaicite:7]{index=7}

## 4.2 Example project `.codex/config.toml`

```toml
# .codex/config.toml

model = "gpt-5.5"
model_reasoning_effort = "high"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
web_search = "cached"
personality = "pragmatic"

project_doc_fallback_filenames = ["CODEX.md", "TEAM_GUIDE.md"]
project_doc_max_bytes = 65536

[agents]
max_threads = 6
max_depth = 1
job_max_runtime_seconds = 1800

[features]
hooks = true

[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
startup_timeout_sec = 20
tool_timeout_sec = 60
enabled = true
default_tools_approval_mode = "prompt"

[mcp_servers.context7.tools.search]
approval_mode = "approve"
````

Important notes:

Codex supports `approval_policy` values such as `untrusted`, `on-request`, `never`, and granular approval objects. `on-failure` is deprecated; OpenAI recommends `on-request` for interactive runs and `never` for non-interactive runs. ([OpenAI Developers][3])

Codex supports `sandbox_mode = "workspace-write"` and named permission profiles such as `:read-only`, `:workspace`, and `:danger-full-access`. ([OpenAI Developers][2])

Project-scoped config cannot override some machine-local or host-owned settings, including model provider, auth, notification, telemetry routing, and profile selection keys. Those belong in user-level config instead. ([OpenAI Developers][3])

---

# 5. Rules: Codex’s command policy mechanism

Claude Code often uses permission allow/deny lists and hooks. Codex has **rules** for controlling which commands can run outside the sandbox.

Rules are experimental, but useful. A `.rules` file lives under a `rules/` folder next to an active config layer, for example:

```text
~/.codex/rules/default.rules
repo/.codex/rules/default.rules
```

Codex scans `rules/` under active config layers at startup, including user rules and trusted project-local rules. When you add a command to the allow list in the TUI, Codex writes to `~/.codex/rules/default.rules`. ([OpenAI Developers][4])

## 5.1 Example `.codex/rules/default.rules`

```text
# Allow safe read-only Git operations outside the sandbox.
prefix_rule(
    pattern = ["git", "status"],
    decision = "allow",
    justification = "Safe read-only repository inspection",
    match = ["git status", "git status --short"],
)

prefix_rule(
    pattern = ["git", "diff"],
    decision = "allow",
    justification = "Safe read-only repository inspection",
    match = ["git diff", "git diff --stat"],
)

# Prompt before GitHub PR reads.
prefix_rule(
    pattern = ["gh", "pr", "view"],
    decision = "prompt",
    justification = "GitHub PR reads may access external workspace information",
    match = [
        "gh pr view 123",
        "gh pr view 123 --json title,body,comments",
    ],
)

# Block dangerous destructive commands.
prefix_rule(
    pattern = ["rm", "-rf"],
    decision = "forbidden",
    justification = "Destructive deletion is not allowed through Codex",
    match = [
        "rm -rf dist",
        "rm -rf /tmp/something",
    ],
)

prefix_rule(
    pattern = ["terraform", "apply"],
    decision = "forbidden",
    justification = "Infrastructure mutation requires human-controlled workflow",
)

prefix_rule(
    pattern = ["terraform", "destroy"],
    decision = "forbidden",
    justification = "Infrastructure destruction must never be agent-triggered",
)

prefix_rule(
    pattern = ["kubectl", "delete"],
    decision = "forbidden",
    justification = "Cluster deletion operations require manual approval outside Codex",
)
```

Rule decisions are:

| Decision    | Meaning                               |
| ----------- | ------------------------------------- |
| `allow`     | Run outside sandbox without prompting |
| `prompt`    | Ask before running                    |
| `forbidden` | Block without prompting               |

If multiple rules match, Codex applies the most restrictive decision: `forbidden` beats `prompt`, and `prompt` beats `allow`. ([OpenAI Developers][4])

Use rules for command-level control. Use hooks for richer validation or lifecycle automation.

---

# 6. Hooks: Codex lifecycle automation

Codex does have hooks. They are not exactly the same as Claude Code hooks, but the concept is similar: deterministic scripts can run during the Codex lifecycle. OpenAI describes hooks as a framework for injecting scripts into the agentic loop, for tasks such as logging, prompt scanning, memory creation, validation checks, and directory-specific prompting. ([OpenAI Developers][5])

Hooks are enabled by default. You can disable them with:

```toml
[features]
hooks = false
```

OpenAI says `hooks` is the canonical key, while `codex_hooks` is a deprecated alias. ([OpenAI Developers][5])

## 6.1 Where Codex looks for hooks

Codex discovers hooks next to active config layers in either:

```text
hooks.json
```

or inline:

```toml
[hooks]
```

Common locations:

```text
~/.codex/hooks.json
~/.codex/config.toml
repo/.codex/hooks.json
repo/.codex/config.toml
```

Plugins can also bundle hooks through their manifest or a default `hooks/hooks.json`. Project-local hooks load only when the project `.codex/` layer is trusted. ([OpenAI Developers][5])

Codex loads all matching hooks from multiple sources. Higher-precedence config layers do not replace lower-precedence hooks. If a single layer contains both `hooks.json` and inline hooks, Codex merges them and warns at startup. ([OpenAI Developers][5])

## 6.2 Hook trust model

Before a non-managed command hook can run, Codex requires review and trust of the exact hook definition. Codex records trust against the hook’s current hash, so changed hooks must be reviewed again. In the CLI, `/hooks` lets you inspect, trust, disable, or review hook sources. Managed hooks are trusted by policy and cannot be disabled through the user hook browser. ([OpenAI Developers][5])

This is an important difference from simple script execution. Codex assumes hooks themselves are powerful and therefore must be trusted.

## 6.3 Supported hook events

Important Codex hook events include:

| Event               | Purpose                                     |
| ------------------- | ------------------------------------------- |
| `SessionStart`      | Add startup context, load notes             |
| `UserPromptSubmit`  | Inspect or transform submitted prompts      |
| `PreToolUse`        | Inspect tool use before execution           |
| `PermissionRequest` | Inspect approval requests                   |
| `PostToolUse`       | Inspect output after tool use               |
| `PreCompact`        | Run before conversation compaction          |
| `PostCompact`       | Run after compaction                        |
| `SubagentStart`     | Add context or policy to a spawned subagent |
| `SubagentStop`      | Inspect subagent completion                 |
| `Stop`              | Final validation or continuation control    |

OpenAI notes that `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PreCompact`, `PostCompact`, `UserPromptSubmit`, `SubagentStop`, and `Stop` run at turn scope, while `SessionStart` and `SubagentStart` run at thread or subagent-start scope. ([OpenAI Developers][5])

## 6.4 Hook config shape

Codex hooks have three levels:

1. Event name, such as `PreToolUse`
2. Matcher group
3. One or more hook handlers

Example `.codex/hooks.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/session_start.py\"",
            "statusMessage": "Loading session notes"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py\"",
            "timeout": 30,
            "statusMessage": "Checking Bash command"
          }
        ]
      }
    ],
    "PermissionRequest": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/permission_request.py\"",
            "timeout": 30,
            "statusMessage": "Checking approval request"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/post_tool_use_review.py\"",
            "timeout": 30,
            "statusMessage": "Reviewing Bash output"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/stop_check.py\"",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

OpenAI recommends resolving repo-local hooks from the Git root because Codex may be started from a subdirectory. ([OpenAI Developers][5])

Important limitations: only `type: "command"` hook handlers run today. `prompt` and `agent` handlers are parsed but skipped. `async` is parsed but async command hooks are not supported yet. ([OpenAI Developers][5])

## 6.5 Equivalent inline TOML

```toml
[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = 'python3 "$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py"'
timeout = 30
statusMessage = "Checking Bash command"

[[hooks.PostToolUse]]
matcher = "^Bash$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = 'python3 "$(git rev-parse --show-toplevel)/.codex/hooks/post_tool_use_review.py"'
timeout = 30
statusMessage = "Reviewing Bash output"
```

This shape is directly supported by Codex. ([OpenAI Developers][5])

## 6.6 Hook output behavior

For `SessionStart`, `PreCompact`, `PostCompact`, `UserPromptSubmit`, `SubagentStop`, and `Stop`, hooks can return JSON fields like:

```json
{
  "continue": true,
  "stopReason": "optional",
  "systemMessage": "optional",
  "suppressOutput": false
}
```

`SessionStart` can output additional context through `hookSpecificOutput.additionalContext`. Plain text on stdout for `SessionStart` is also added as extra developer context. ([OpenAI Developers][5])

Caution: `PreToolUse` and `PermissionRequest` currently support `systemMessage`, but not `continue`, `stopReason`, or `suppressOutput`. If a `PreToolUse` hook returns unsupported fields, Codex marks the hook as failed, reports the error, and continues the tool call. ([OpenAI Developers][5])

That means blocking tool execution should primarily use **rules**, sandbox/approval policy, and permission settings, not unsupported `PreToolUse` output fields.

---

# 7. Example hook scripts

## 7.1 Prompt scanner

`.codex/hooks/user_prompt_submit_data_flywheel.py`

```python
#!/usr/bin/env python3
import json
import re
import sys

payload = json.load(sys.stdin)
prompt = payload.get("prompt", "")

patterns = [
    r"sk-[A-Za-z0-9_\-]{20,}",
    r"AKIA[0-9A-Z]{16}",
    r"-----BEGIN (RSA|OPENSSH|EC|PRIVATE) KEY-----",
]

for pattern in patterns:
    if re.search(pattern, prompt):
        print(json.dumps({
            "continue": False,
            "stopReason": "Possible secret detected in prompt.",
            "systemMessage": "The prompt appears to contain a secret or credential. Remove it before continuing."
        }))
        sys.exit(0)

sys.exit(0)
```

Use this for prompt-time safety. It is safer than hoping `AGENTS.md` instructions prevent accidental secret leakage.

## 7.2 Post-tool review

`.codex/hooks/post_tool_use_review.py`

```python
#!/usr/bin/env python3
import json
import sys

payload = json.load(sys.stdin)

tool_name = payload.get("tool_name", "")
tool_output = payload.get("tool_output", "") or ""

if tool_name == "Bash" and "Traceback" in tool_output:
    print(json.dumps({
        "continue": True,
        "systemMessage": "The last command produced a Python traceback. Inspect the failure before continuing."
    }))

sys.exit(0)
```

Use this to add deterministic reminders or warnings after commands.

---

# 8. Skills in Codex

A Codex skill is a directory with a `SKILL.md` file plus optional scripts, references, assets, and `agents/openai.yaml`. The `SKILL.md` file must include `name` and `description`. Skills are progressively disclosed: Codex initially sees skill names, descriptions, and paths; it reads the full `SKILL.md` only when it selects the skill. ([OpenAI Developers][6])

Skills are available in Codex CLI, IDE extension, and Codex app. Codex can activate them explicitly, for example through `/skills` or `$skill-name`, or implicitly when a task matches the skill description. ([OpenAI Developers][6])

## 8.1 Where Codex reads skills from

Codex scans skills from repository, user, admin, and system locations. For repositories, it scans `.agents/skills` from the current working directory up to the repository root. User skills live under `$HOME/.agents/skills`; admin skills live under `/etc/codex/skills`; system skills are bundled with Codex. ([OpenAI Developers][6])

Important: Codex skills use `.agents/skills`, not `.codex/skills`.

Recommended repo structure:

```text
repo/
└── .agents/
    └── skills/
        ├── review-pr/
        │   ├── SKILL.md
        │   └── references/
        │       └── review-checklist.md
        ├── debug-failing-test/
        │   └── SKILL.md
        └── write-adr/
            ├── SKILL.md
            └── assets/
                └── adr-template.md
```

## 8.2 Skill template

````md
---
name: review-pr
description: Review the current branch or pull request for correctness, security, maintainability, regressions, and missing tests. Use when the user asks for PR review, code review, pre-merge review, or risk review.
---

# Review PR skill

Review the current change like a senior maintainer.

## Workflow

1. Inspect the changed files.
2. Understand the intended behavior.
3. Read surrounding code only where needed.
4. Identify correctness risks first.
5. Identify security, maintainability, performance, and test gaps.
6. Suggest minimal fixes before larger refactors.

## Commands

Prefer:

```bash
git status --short
git diff --stat
git diff main...HEAD
````

If the base branch is not `main`, infer it from the current branch or ask for clarification.

## Output format

Return:

1. Summary
2. High-risk issues
3. Medium-risk issues
4. Test gaps
5. Suggested next actions
6. Confidence level

````

## 8.3 Skill with scripts and references

```text
.agents/skills/release-check/
├── SKILL.md
├── references/
│   └── release-policy.md
├── scripts/
│   └── collect-release-info.sh
└── assets/
    └── release-notes-template.md
````

`SKILL.md`:

````md
---
name: release-check
description: Prepare a release readiness review using repository state, tests, changelog, dependency changes, and release policy. Use before tagging or preparing release notes.
---

# Release readiness review

Use `references/release-policy.md` as the source of truth for release criteria.

Run the helper script only if the user asks for a concrete release review:

```bash
./.agents/skills/release-check/scripts/collect-release-info.sh
````

Check:

1. Changelog completeness
2. Version bump
3. Test status
4. Dependency changes
5. Migration or rollback notes
6. Operational risks
7. Security-sensitive changes

````

## 8.4 When to use a skill

Use a skill when the work is:

- Repeated often
- Procedural
- Repo/team-specific
- Too long for `AGENTS.md`
- Useful across sessions
- Useful as a repeatable workflow

Examples:

```text
review-pr
implement-feature
debug-failing-test
write-adr
create-release-notes
audit-terraform-change
explain-module
generate-test-plan
````

Do not use a skill for always-on rules like “do not read `.env`”. Use rules, config, sandbox, and hooks for that.

---

# 9. Subagents in Codex

Codex supports subagent workflows. Current Codex releases enable subagent workflows by default. Codex can spawn specialized agents in parallel and consolidate their results, which is useful for parallel tasks such as codebase exploration, PR review dimensions, and multi-step implementation planning. ([OpenAI Developers][7])

Important difference from Claude Code: Codex only spawns subagents when you explicitly ask it to. It does not automatically route to subagents purely because a matching custom agent exists. ([OpenAI Developers][7])

## 9.1 Built-in Codex agents

Codex ships with built-in agents:

| Agent      | Purpose                                    |
| ---------- | ------------------------------------------ |
| `default`  | General-purpose fallback                   |
| `worker`   | Execution-focused implementation/fix agent |
| `explorer` | Read-heavy codebase exploration agent      |

([OpenAI Developers][7])

## 9.2 Custom agent location

Custom agents are standalone TOML files:

```text
~/.codex/agents/reviewer.toml
repo/.codex/agents/reviewer.toml
```

Each file defines one custom agent. Required fields are:

```toml
name = "..."
description = "..."
developer_instructions = """
...
"""
```

Optional fields include `nickname_candidates`, `model`, `model_reasoning_effort`, `sandbox_mode`, `mcp_servers`, and `skills.config`, inherited from the parent session if omitted. ([OpenAI Developers][7])

## 9.3 Project subagent config

`.codex/config.toml`:

```toml
[agents]
max_threads = 6
max_depth = 1
job_max_runtime_seconds = 1800
```

OpenAI documents these fields for global subagent settings: `agents.max_threads`, `agents.max_depth`, and `agents.job_max_runtime_seconds`. ([OpenAI Developers][7])

## 9.4 Example custom agents

### `.codex/agents/pr-explorer.toml`

```toml
name = "pr_explorer"
description = "Read-only codebase explorer for gathering evidence before changes are proposed."
model = "gpt-5.5"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"

developer_instructions = """
Stay in exploration mode.

Trace the real execution path.
Cite files, symbols, and assumptions.
Avoid proposing fixes unless the parent agent asks for them.
Prefer fast search and targeted file reads over broad scans.
"""
```

### `.codex/agents/reviewer.toml`

```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, regressions, maintainability, and missing tests."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
nickname_candidates = ["Atlas", "Delta", "Echo"]

developer_instructions = """
Review code like a senior maintainer.

Prioritize:
- correctness
- security
- behavior regressions
- missing test coverage
- maintainability
- API compatibility

Do not edit files.
Return concrete findings with severity, evidence, and suggested remediation.
"""
```

### `.codex/agents/test-engineer.toml`

```toml
name = "test_engineer"
description = "Designs or updates tests for a change using existing project test patterns."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You are a test engineer.

Your job:
- Identify missing test cases.
- Prefer focused unit tests before broad integration tests.
- Reuse existing test utilities and patterns.
- Avoid unrelated test rewrites.
- Run the smallest relevant test first.
- Summarize tests added, tests run, and residual risk.
"""
```

### `.codex/agents/security-reviewer.toml`

```toml
name = "security_reviewer"
description = "Reviews auth, secrets, dependency, infrastructure, input validation, and data exposure risks."
model = "gpt-5.5"
model_reasoning_effort = "xhigh"
sandbox_mode = "read-only"

developer_instructions = """
You are a security reviewer.

Focus on:
- secrets exposure
- unsafe shell commands
- authorization gaps
- authentication flows
- injection risks
- dependency risk
- infrastructure misconfiguration
- data leakage
- unsafe logging

Do not edit files.
Return severity, evidence, exploitability, and remediation.
"""
```

## 9.5 How to ask Codex to use subagents

Example prompt:

```text
Review this branch against main. Spawn one subagent for each area, wait for all of them, and summarize findings:

1. Security risks
2. Correctness bugs
3. Test gaps
4. Maintainability
5. Operational or DevOps risks
```

Codex handles orchestration, waits for results, and returns a consolidated response. ([OpenAI Developers][7])

## 9.6 Subagent approval and sandbox behavior

Subagents inherit the current sandbox policy. Approval requests can surface from inactive subagent threads in interactive CLI sessions. In non-interactive flows, if a subagent needs fresh approval and cannot surface it, the action fails and Codex reports the error back to the parent workflow. Runtime overrides such as `/permissions` changes or `--yolo` are also reapplied to child agents. ([OpenAI Developers][7])

---

# 10. Plugins in Codex

Codex plugins bundle reusable workflows and integrations. OpenAI describes plugins as packages that can contain skills, app integrations, and MCP servers. ([OpenAI Developers][8])

A plugin can contain:

| Component            | Purpose                                                      |
| -------------------- | ------------------------------------------------------------ |
| Skills               | Reusable instructions and workflows                          |
| Apps                 | Connections to tools like GitHub, Slack, Google Drive, Gmail |
| MCP servers          | External tools or shared information sources                 |
| Hooks                | Lifecycle automation packaged with the plugin                |
| Marketplace metadata | Distribution and install policy                              |

Use plugins when you want to distribute a stable workflow across teams or repositories. OpenAI explicitly recommends starting with a local skill if you are still iterating on one repo or personal workflow, and building a plugin when you want to share the workflow across teams, bundle app/MCP config, package hooks, or publish a stable package. ([OpenAI Developers][9])

## 10.1 Plugin structure

```text
plugins/team-engineering-plugin/
├── .codex-plugin/
│   └── plugin.json
├── skills/
│   ├── review-pr/
│   │   └── SKILL.md
│   └── write-adr/
│       └── SKILL.md
├── hooks/
│   └── hooks.json
├── mcp/
│   └── README.md
└── README.md
```

## 10.2 Minimal plugin manifest

`plugins/team-engineering-plugin/.codex-plugin/plugin.json`

```json
{
  "name": "team-engineering-plugin",
  "version": "1.0.0",
  "description": "Reusable engineering workflows for PR review, test design, ADR writing, and release checks.",
  "skills": "./skills/"
}
```

OpenAI’s minimal plugin example uses `.codex-plugin/plugin.json` with fields such as `name`, `version`, `description`, and `skills`. Codex uses the plugin name as the plugin identifier and namespace. ([OpenAI Developers][9])

## 10.3 Plugin with hooks

```json
{
  "name": "team-engineering-plugin",
  "version": "1.0.0",
  "description": "Shared engineering workflows and lifecycle checks.",
  "skills": "./skills/",
  "hooks": "./hooks/hooks.json"
}
```

`hooks/hooks.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"${CODEX_PLUGIN_ROOT}/hooks/post_bash_review.py\"",
            "timeout": 30,
            "statusMessage": "Running plugin Bash review"
          }
        ]
      }
    ]
  }
}
```

Plugin hooks load alongside other hook sources and follow the same trust-review flow as non-managed hooks. ([OpenAI Developers][5])

## 10.4 Repo-local plugin marketplace

For repo-scoped plugins, add:

```text
repo/.agents/plugins/marketplace.json
repo/plugins/team-engineering-plugin/
```

Marketplace example:

```json
{
  "name": "local-repo",
  "interface": {
    "displayName": "Local Repo Plugins"
  },
  "plugins": [
    {
      "name": "team-engineering-plugin",
      "source": {
        "source": "local",
        "path": "./plugins/team-engineering-plugin"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

OpenAI documents `$REPO_ROOT/.agents/plugins/marketplace.json` for repo-scoped plugin lists and explains that `source.path` points to the plugin directory relative to the marketplace root. ([OpenAI Developers][9])

You can also add a marketplace with:

```bash
codex plugin marketplace add ./local-marketplace-root
codex plugin marketplace list
codex plugin marketplace upgrade
codex plugin marketplace remove marketplace-name
```

Marketplace sources can be GitHub shorthand, Git URLs, SSH Git URLs, HTTP URLs, or local marketplace roots. ([OpenAI Developers][9])

---

# 11. MCP in Codex

MCP gives Codex access to third-party tools and context, such as documentation, browser tools, Figma, GitHub, Sentry, and internal systems. Codex supports MCP servers in the CLI and IDE extension. ([OpenAI Developers][10])

Codex supports:

| MCP transport           | Support                                                         |
| ----------------------- | --------------------------------------------------------------- |
| STDIO servers           | Local process launched by command                               |
| Streamable HTTP servers | Remote server over HTTP                                         |
| Env vars                | Supported                                                       |
| Bearer token auth       | Supported                                                       |
| OAuth                   | Supported through `codex mcp login <server-name>`               |
| Server instructions     | Codex reads MCP `instructions` and uses them as server guidance |

([OpenAI Developers][10])

## 11.1 MCP config example

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env_vars = ["LOCAL_TOKEN"]
startup_timeout_sec = 20
tool_timeout_sec = 45
enabled = true
default_tools_approval_mode = "prompt"

[mcp_servers.context7.env]
MY_ENV_VAR = "MY_ENV_VALUE"

[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
http_headers = { "X-Figma-Region" = "us-east-1" }

[mcp_servers.chrome_devtools]
url = "http://localhost:3000/mcp"
enabled_tools = ["open", "screenshot"]
disabled_tools = ["screenshot"]
default_tools_approval_mode = "prompt"
startup_timeout_sec = 20
tool_timeout_sec = 45
enabled = true

[mcp_servers.chrome_devtools.tools.open]
approval_mode = "approve"
```

Codex config supports `enabled_tools`, `disabled_tools`, `default_tools_approval_mode`, and per-tool approval settings. `disabled_tools` is applied after `enabled_tools`. ([OpenAI Developers][10])

---

# 12. Skill vs plugin vs subagent vs hook vs rule

## 12.1 Comparison table

| Mechanism            | Codex location                                         | Best for                                     | Triggered how                          | Should it be committed?            |
| -------------------- | ------------------------------------------------------ | -------------------------------------------- | -------------------------------------- | ---------------------------------- |
| `AGENTS.md`          | Repo root or nested dirs                               | Always-on repo guidance                      | Auto-loaded at session start           | Yes                                |
| `.codex/config.toml` | `.codex/config.toml`                                   | Project config, sandbox, MCP, model defaults | Auto-loaded in trusted project         | Sometimes, if safe                 |
| Rules                | `.codex/rules/*.rules`                                 | Command allow/prompt/block policy            | Command approval flow                  | Yes, if team policy                |
| Hooks                | `.codex/hooks.json` or `[hooks]`                       | Lifecycle scripts, validation, logging       | Codex lifecycle events                 | Yes, if safe and reviewed          |
| Skill                | `.agents/skills/*/SKILL.md`                            | Repeatable workflow                          | Explicit `$skill` or implicit matching | Yes                                |
| Subagent             | `.codex/agents/*.toml`                                 | Parallel specialist worker                   | Explicit request to spawn agents       | Yes                                |
| Plugin               | `plugins/*/.codex-plugin/plugin.json` plus marketplace | Reusable package for teams/repos             | Installed from plugin marketplace      | Yes, if intended for repo          |
| MCP                  | `config.toml`                                          | External tools and data                      | Tool use                               | Depends on secrets and environment |

## 12.2 Decision rules

Use **`AGENTS.md`** when the instruction is always relevant:

```text
Use existing architecture.
Do not commit unless asked.
Run lint and tests before completion.
Never read secrets.
```

Use **rules** when you need command-level policy:

```text
Allow git status.
Prompt for gh pr view.
Block terraform destroy.
Block kubectl delete.
```

Use **hooks** when you need lifecycle automation:

```text
Scan prompts for secrets.
Add startup context.
Review Bash output.
Run final stop checks.
Log sessions to internal analytics.
```

Use **skills** when you need reusable workflows:

```text
Review PR.
Debug failing test.
Write ADR.
Prepare release notes.
Audit Terraform change.
```

Use **subagents** when work should happen in parallel or in isolated specialist threads:

```text
Security review.
Test gap review.
Architecture review.
Docs review.
Codebase exploration.
```

Use **plugins** when a workflow is stable and should be distributed:

```text
Team engineering plugin.
Company security plugin.
Repo-specific plugin marketplace.
Shared GitHub/Sentry/Figma MCP bundle.
```

Use **MCP** when Codex needs external system access:

```text
Figma design inspection.
Sentry logs.
GitHub PR metadata.
Documentation search.
Internal service APIs.
```

---

# 13. Recommended setup for your professional engineering repos

For your kind of work, especially cloud, DevOps, AI agents, React, and enterprise product engineering, I would use this structure:

```text
repo/
├── AGENTS.md
├── .codex/
│   ├── config.toml
│   ├── hooks.json
│   ├── rules/
│   │   └── default.rules
│   ├── agents/
│   │   ├── pr-explorer.toml
│   │   ├── reviewer.toml
│   │   ├── test-engineer.toml
│   │   ├── security-reviewer.toml
│   │   └── devops-reviewer.toml
│   └── hooks/
│       ├── user_prompt_secret_scan.py
│       ├── post_tool_use_review.py
│       └── stop_check.py
├── .agents/
│   ├── skills/
│   │   ├── implement-feature/
│   │   │   └── SKILL.md
│   │   ├── review-pr/
│   │   │   └── SKILL.md
│   │   ├── debug-failing-test/
│   │   │   └── SKILL.md
│   │   ├── write-adr/
│   │   │   └── SKILL.md
│   │   ├── audit-terraform-change/
│   │   │   └── SKILL.md
│   │   └── release-check/
│   │       └── SKILL.md
│   └── plugins/
│       └── marketplace.json
└── plugins/
    └── team-engineering-plugin/
```

Start with:

```text
AGENTS.md
.codex/config.toml
.codex/rules/default.rules
.agents/skills/review-pr/SKILL.md
.agents/skills/debug-failing-test/SKILL.md
.codex/agents/reviewer.toml
.codex/agents/security-reviewer.toml
```

Then add hooks and plugins only after the workflow stabilizes.

---

# 14. Practical repo templates

## 14.1 `.codex/config.toml`

```toml
model = "gpt-5.5"
model_reasoning_effort = "high"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
web_search = "cached"
personality = "pragmatic"

project_doc_fallback_filenames = ["CODEX.md", "TEAM_GUIDE.md"]
project_doc_max_bytes = 65536

[features]
hooks = true

[agents]
max_threads = 6
max_depth = 1
job_max_runtime_seconds = 1800

[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
enabled = true
startup_timeout_sec = 20
tool_timeout_sec = 60
default_tools_approval_mode = "prompt"

[mcp_servers.context7.tools.search]
approval_mode = "approve"
```

## 14.2 `.codex/rules/default.rules`

```text
prefix_rule(
    pattern = ["git", "status"],
    decision = "allow",
    justification = "Read-only repository inspection",
)

prefix_rule(
    pattern = ["git", "diff"],
    decision = "allow",
    justification = "Read-only repository inspection",
)

prefix_rule(
    pattern = ["npm", "test"],
    decision = "allow",
    justification = "Project test command",
)

prefix_rule(
    pattern = ["npm", "run", "lint"],
    decision = "allow",
    justification = "Project lint command",
)

prefix_rule(
    pattern = ["gh", "pr", "view"],
    decision = "prompt",
    justification = "External GitHub access should be reviewed",
)

prefix_rule(
    pattern = ["terraform", "apply"],
    decision = "forbidden",
    justification = "Infrastructure mutation must not be agent-triggered",
)

prefix_rule(
    pattern = ["terraform", "destroy"],
    decision = "forbidden",
    justification = "Infrastructure destruction is blocked",
)

prefix_rule(
    pattern = ["kubectl", "delete"],
    decision = "forbidden",
    justification = "Cluster deletion is blocked",
)

prefix_rule(
    pattern = ["rm", "-rf"],
    decision = "prompt",
    justification = "Recursive deletion requires review",
)
```

## 14.3 `.codex/hooks.json`

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/user_prompt_secret_scan.py\"",
            "timeout": 10,
            "statusMessage": "Scanning prompt for secrets"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/post_tool_use_review.py\"",
            "timeout": 30,
            "statusMessage": "Reviewing command output"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/stop_check.py\"",
            "timeout": 30,
            "statusMessage": "Running final completion check"
          }
        ]
      }
    ]
  }
}
```

## 14.4 `.agents/skills/implement-feature/SKILL.md`

```md
---
name: implement-feature
description: Implement a feature using existing architecture, minimal diffs, relevant tests, and a final risk summary. Use when the user asks to add or update functionality.
---

# Implement feature

## Goal

Implement the requested feature with minimal, maintainable changes.

## Rules

- Inspect existing patterns first.
- Prefer small diffs.
- Do not introduce new dependencies unless necessary.
- Preserve public APIs unless the requested change requires otherwise.
- Add or update tests.
- Do not commit or push.

## Workflow

1. Identify relevant files.
2. Explain the implementation plan briefly.
3. Make the smallest useful change.
4. Run the smallest relevant test.
5. Run lint/typecheck if applicable.
6. Summarize changed files, tests run, and residual risks.
```

## 14.5 `.agents/skills/audit-terraform-change/SKILL.md`

```md
---
name: audit-terraform-change
description: Review Terraform or infrastructure-as-code changes for safety, drift risk, blast radius, secret exposure, rollback, and operational impact.
---

# Audit Terraform change

Review infrastructure changes conservatively.

## Inspect

- Changed Terraform modules
- Variables
- Outputs
- Providers
- State-related assumptions
- Environment overlays
- Secret references
- IAM changes
- Network/security group changes

## Never do automatically

- Do not run `terraform apply`.
- Do not run `terraform destroy`.
- Do not approve production changes.
- Do not rotate or expose secrets.

## Output

1. Summary
2. Blast radius
3. Security risks
4. Operational risks
5. Rollback strategy
6. Required manual validation
7. Suggested safer alternative, if any
```

---

# 15. How Codex differs from Claude Code

| Area                    | Claude Code                                       | Codex                                                    |
| ----------------------- | ------------------------------------------------- | -------------------------------------------------------- |
| Main instruction file   | `CLAUDE.md`                                       | `AGENTS.md`                                              |
| Project config          | `.claude/settings.json`                           | `.codex/config.toml`                                     |
| Skills location         | `.claude/skills`                                  | `.agents/skills`                                         |
| Subagent definition     | Markdown with frontmatter                         | TOML files in `.codex/agents`                            |
| Subagent triggering     | Can auto-route based on description in many flows | Explicitly ask Codex to spawn agents                     |
| Hooks                   | JSON settings and hook handlers                   | `hooks.json` or inline TOML, command handlers only today |
| Command policy          | Permissions and hooks                             | Sandbox, approval policy, rules, hooks                   |
| Plugin manifest         | `.claude-plugin/plugin.json`                      | `.codex-plugin/plugin.json`                              |
| Repo plugin marketplace | Different plugin mechanism                        | `.agents/plugins/marketplace.json`                       |
| MCP config              | Commonly config/settings                          | `config.toml` under `[mcp_servers.*]`                    |

The architectural lesson: **Claude Code leans heavily on `.claude` artifacts; Codex splits repo instructions into `AGENTS.md`, capabilities into `.agents`, and runtime configuration into `.codex`.**

---

# 16. Best-practice rollout plan

## Phase 1: Basic repo readiness

Add:

```text
AGENTS.md
.codex/config.toml
.codex/rules/default.rules
```

Goal: make Codex understand the project, work safely, and avoid destructive commands.

## Phase 2: Reusable engineering workflows

Add:

```text
.agents/skills/review-pr/SKILL.md
.agents/skills/debug-failing-test/SKILL.md
.agents/skills/implement-feature/SKILL.md
.agents/skills/write-adr/SKILL.md
```

Goal: stop retyping the same prompts.

## Phase 3: Specialist subagents

Add:

```text
.codex/agents/pr-explorer.toml
.codex/agents/reviewer.toml
.codex/agents/security-reviewer.toml
.codex/agents/test-engineer.toml
.codex/agents/devops-reviewer.toml
```

Goal: parallel reviews and isolated specialist work.

## Phase 4: Hooks

Add:

```text
.codex/hooks.json
.codex/hooks/user_prompt_secret_scan.py
.codex/hooks/post_tool_use_review.py
.codex/hooks/stop_check.py
```

Goal: deterministic lifecycle checks.

## Phase 5: Plugin packaging

Add:

```text
plugins/team-engineering-plugin/.codex-plugin/plugin.json
.agents/plugins/marketplace.json
```

Goal: reuse across repos or teams.

---

# 17. My recommended default

For your use case, I would not begin with a plugin. I would start with a **repo-local Codex operating system**:

```text
AGENTS.md
.codex/config.toml
.codex/rules/default.rules
.agents/skills/
.codex/agents/
```

Then, after you reuse the same setup in two or three repositories, promote the stable parts into a plugin.

Recommended default structure:

```text
repo/
├── AGENTS.md
├── .codex/
│   ├── config.toml
│   ├── hooks.json
│   ├── rules/default.rules
│   └── agents/
│       ├── reviewer.toml
│       ├── security-reviewer.toml
│       └── test-engineer.toml
└── .agents/
    └── skills/
        ├── review-pr/
        ├── debug-failing-test/
        ├── implement-feature/
        └── write-adr/
```

This gives you a clean separation:

```text
AGENTS.md       = project knowledge
config.toml     = runtime behavior
rules           = command policy
hooks           = deterministic automation
skills          = reusable workflows
subagents       = specialist workers
plugins         = distribution packaging
MCP             = external tools and systems
```

That is the structure I would use for a serious Codex-enabled software engineering repository.

[1]: https://developers.openai.com/codex/guides/agents-md "Custom instructions with AGENTS.md – Codex | OpenAI Developers"
[2]: https://developers.openai.com/codex/config-basic "Config basics – Codex | OpenAI Developers"
[3]: https://developers.openai.com/codex/config-reference "Configuration Reference – Codex | OpenAI Developers"
[4]: https://developers.openai.com/codex/rules "Rules – Codex | OpenAI Developers"
[5]: https://developers.openai.com/codex/hooks "Hooks – Codex | OpenAI Developers"
[6]: https://developers.openai.com/codex/skills "Agent Skills – Codex | OpenAI Developers"
[7]: https://developers.openai.com/codex/subagents "Subagents – Codex | OpenAI Developers"
[8]: https://developers.openai.com/codex/plugins "Plugins – Codex | OpenAI Developers"
[9]: https://developers.openai.com/codex/plugins/build "Build plugins – Codex | OpenAI Developers"
[10]: https://developers.openai.com/codex/mcp "Model Context Protocol – Codex | OpenAI Developers"
