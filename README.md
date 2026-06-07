# Prepare Your Repo for AI Agents

A practical, platform-neutral knowledge base for structuring repositories so AI coding agents can understand, follow, and enforce your project's standards.

**Live site:** [rafflesiakhan.github.io/prepare-repo-for-ai-agent](https://rafflesiakhan.github.io/prepare-repo-for-ai-agent)

---

## The problem this solves

When you work across multiple AI agent sessions, context grows, memory gets stale, and agents start making mistakes when creating skills, rules, or reusable components. This guide gives agents a stable reference — a style guide they can look up whenever a project needs a new skill, rule, hook, or plugin.

---

## Documents

### [Terminology.md](./Terminology.md) — Platform-neutral taxonomy

A 27-section map of all agent building blocks, usable across Claude Code, Codex, IBM Bob, Cursor, and other agents:

- Metadata files, rules, skills, workflows, subagents, hooks, guardrails, plugins, MCP servers
- Decision trees: which building block to use when
- Common mistakes and how to avoid them
- Recommended repository layouts (minimal and mature)
- Platform mapping table (Claude Code / Codex / IBM Bob)

Start here if you are unsure what kind of component to create.

### [Claude-Style.md](./Claude-Style.md) — Claude Code reference

Complete setup guide for Claude Code projects:

- CLAUDE.md hierarchy, templates, and what to put in it
- `.claude/settings.json` with permission rules and all config options
- All 30 hook lifecycle events, 4 handler types, I/O contract, and environment variables
- Ready-to-use skill templates (review-pr, implement-feature, debug-failing-test, write-adr)
- Ready-to-use subagent templates (code-reviewer, repo-architect, test-engineer, security-reviewer, devops-reviewer)
- MCP integration, security patterns, anti-patterns, and a phase-by-phase rollout plan

### [Codex-Style.md](./Codex-Style.md) — Codex reference

Setup patterns for Codex and OpenAI-compatible agents. AGENTS.md structure, skills, subagents, configuration, and Codex-specific decision trees.

### [IBMBob-Style.md](./IBMBob-Style.md) — IBM Bob reference

Configuration and mental models for IBM Bob. Custom modes, rules, skills, .bob/ directory structure, and IBM Bob–specific patterns.

---

## Mental model

```
Metadata file  → What the agent should KNOW
Rules          → How the agent should BEHAVE
Skill          → How to do REPEATED TASKS
Workflow       → The PROCESS from start to finish
Subagent       → WHO handles specialized work
Hook           → What MUST happen automatically
Guardrail      → What the agent CANNOT do
Plugin         → How to SHARE capabilities across repos
```

---

## Quick start

**Starting a new project:**
1. Read sections 1–3 of [Terminology.md](./Terminology.md) (taxonomy, metadata files, rules) — 10 min
2. Pick your agent tool and read its guide
3. Start with the minimal setup: one metadata file + one rule file
4. Add skills as you find yourself pasting the same instructions repeatedly

**Migrating an existing project:**
1. Run through the decision tree in [Terminology.md](./Terminology.md#24-practical-decision-tree)
2. Start with your tool's guide, implement one layer at a time
3. Don't add everything at once — skills, hooks, and subagents on demand

**Designing a team-wide setup:**
1. Read [Terminology.md](./Terminology.md) section 25 (recommended repository layout)
2. Read section 27 (recommended default strategy for layering)
3. Start repo-local; promote to plugin only after the same setup is reused across 2–3 repos

---

## Platform mapping

| Concept | Claude Code | Codex | IBM Bob |
|---|---|---|---|
| Metadata | `CLAUDE.md` | `AGENTS.md` | `AGENTS.md` |
| Rules | `.claude/rules/` | `.codex/rules/` | `.bob/rules/` |
| Skills | `.claude/skills/` | `.agents/skills/` | `.bob/skills/` |
| Config | `.claude/settings.json` | `.codex/config.toml` | `.bob/custom_modes.yaml` |
| MCP | `.mcp.json` | `.mcp.json` | `.mcp.json` |

---

## Contributing

Found a mistake or have a working pattern worth documenting? Open an issue or PR. This is a living document — real-world patterns make it better.

---

## License

Provided as-is for public use. Share, remix, and adapt freely.
