# 🤖 Prepare Your Repo for AI Agents

**A practical, platform-neutral guide to preparing your repository for modern AI coding agents—Claude Code, Codex, IBM Bob, Cursor, and beyond.**

Are you starting a new AI-agent-ready project? Or migrating an existing repo to work seamlessly with AI assistants? This repository provides **battle-tested patterns, terminology, and decision trees** for structuring projects so that AI agents can understand, follow, and enforce your project's unique needs.

---

## 🎯 What This Solves

AI coding agents work best when they understand:

- **What to build** (project context)
- **How to build it** (coding standards, testing policy, security rules)
- **What they can access** (tools, APIs, external systems)
- **How to repeat common tasks** (workflows, checklists, decision trees)
- **What must never happen** (safety guardrails, enforced rules)

This guide shows you how to structure metadata files, skills, rules, hooks, subagents, and more—**platform-agnostic language that works across Claude Code, Codex, IBM Bob, and internal enterprise agents**.

---

## 📚 What's Included

### **[Terminology.md](./Terminology.md)** — The Complete Map
A comprehensive, 27-section taxonomy of all building blocks used by modern AI agents:

- **Metadata files** — persistent project context (AGENTS.md, CLAUDE.md, GEMINI.md)
- **Rules & custom instructions** — behavioral constraints
- **Skills & workflows** — reusable, repeatable task procedures
- **Subagents & custom modes** — specialized workers and behavior profiles
- **Hooks & guardrails** — deterministic safety and automation
- **Plugins & extensions** — packaged, shareable capabilities
- **Templates & output schemas** — consistent output formats
- **MCP servers** — external tool bridges
- **Memory & evaluation** — persistent context and quality gates
- **Decision trees** — how to choose the right tool for your use case

**Use this when:** you're unsure whether to use a skill, hook, rule, or metadata file. It explains what each thing is, when to use it, and when *not* to use it.

### **[Calude-Style.md](./Calude-Style.md)** — Claude Code Deep Dive
Complete reference for Claude Code–specific setup:

- Mental models for how Claude reads from your repo
- Recommended directory structure (.claude/, CLAUDE.md, settings.json)
- Concrete examples of CLAUDE.md templates
- How to write effective rules and skills for Claude
- Hook examples (block dangerous commands, run formatters, validate secrets)
- Settings.json configuration patterns
- Common Claude mistakes and how to avoid them

**Use this when:** you're using Claude Code and want specific, working examples.

### **[Codex-Style.md](./Codex-Style.md)** — Codex & OpenAI Agents
Complete reference for Codex and OpenAI-compatible agents:

- Codex-specific setup patterns
- AGENTS.md structure and content
- Codex skills, subagents, and plugin models
- Configuration for Codex workflows
- Decision trees for Codex-specific choices

**Use this when:** you're using OpenAI's Codex or compatible agents.

### **[IBMBob-Style.md](./IBMBob-Style.md)** — IBM Bob Configuration
Complete reference for IBM Bob:

- IBM Bob mental models and configuration
- Custom modes and rules
- Skills and workflow setup
- .bob/ directory structure
- IBM Bob–specific patterns and examples

**Use this when:** you're using IBM Bob.

---

## 🚀 Quick Start: Choose Your Path

### **I'm starting a brand-new project**
1. Read the **"Recommended default strategy"** in [Terminology.md](./Terminology.md#27-recommended-default-strategy)
2. Pick your AI agent tool (Claude Code / Codex / IBM Bob / other)
3. Follow the tool-specific guide (Calude-Style.md / Codex-Style.md / IBMBob-Style.md)
4. Use the **"Minimal starting setup"** as your template

### **I'm migrating an existing project**
1. Run through the **"Practical decision tree"** in [Terminology.md](./Terminology.md#24-practical-decision-tree)
2. Check the **"Common mistakes"** section to avoid pitfalls
3. Start with your tool's guide and implement one layer at a time
4. Don't do everything at once—add skills, rules, and hooks as you find gaps

### **I need to review a specific concept**
1. Find it in the **[Terminology.md](./Terminology.md)** table of contents
2. Read the "When to use" and "When not to use" sections
3. See decision trees and examples
4. Jump to your tool's guide for working code/config examples

### **I'm designing a team-wide agent setup**
1. Start with **[Terminology.md](./Terminology.md#25-recommended-repository-layout-for-a-mature-ai-agent-ready-repo)** section 25 (Recommended repository layout)
2. Review **section 27** (Recommended default strategy) for layering
3. Follow your tool's guide for implementation
4. Read **"Common mistakes"** to avoid over-engineering

---

## 📖 The Mental Model (TL;DR)

```
Metadata file  → What the agent should KNOW
Rules          → How the agent should BEHAVE
Prompt         → What I want RIGHT NOW
Skill          → How to do REPEATED TASKS
Workflow       → The PROCESS from start to finish
Subagent       → WHO should handle specialized work
Tool           → What the agent CAN DO
Hook           → What MUST happen automatically
Guardrail      → What the agent CANNOT do
Plugin         → How to SHARE capabilities across repos
```

---

## 🎓 How to Read This Repo

**Start here:** [Terminology.md](./Terminology.md)  
→ Read sections 1–3 (taxonomy, metadata files, rules) — 10 min  
→ Jump to your tool's guide for examples (Calude-Style.md / Codex-Style.md / IBMBob-Style.md)  

**Deep dive:** Read all 28 sections in [Terminology.md](./Terminology.md) for the complete mental model

**Design mode:** Use the **decision trees** and **common mistakes** sections to avoid pitfalls

**Troubleshoot:** If your agent isn't following instructions, check [Terminology.md](./Terminology.md#16-guardrails-permissions-and-sandboxing) section 16 (Guardrails vs. rules) and section 18 (Hooks)

---

## ✅ What You'll Know After Reading

- **What each building block does** — metadata files, skills, rules, hooks, subagents, plugins, MCP servers
- **When to use each one** — decision trees to avoid over-engineering
- **How to structure your repo** — recommended layouts for small and mature projects
- **Tool-specific patterns** — working examples for Claude Code, Codex, IBM Bob
- **Common pitfalls** — what breaks and why
- **How to layer capabilities** — start simple, add complexity only when needed

---

## 🔗 Platform Mapping

| Concept | Claude Code | Codex | IBM Bob |
|---------|------------|-------|---------|
| Metadata | `CLAUDE.md` | `AGENTS.md` | `AGENTS.md` |
| Rules | `.claude/rules/` | `.codex/rules/` | `.bob/rules/` |
| Skills | `.claude/skills/` | `.agents/skills/` | `.bob/skills/` |
| Config | `.claude/settings.json` | `.codex/config.toml` | `.bob/custom_modes.yaml` |
| MCP | `.mcp.json` | `.mcp.json` | `.mcp.json` |

See [Terminology.md](./Terminology.md#25-recommended-repository-layout-for-a-mature-ai-agent-ready-repo) section 25 for the complete mapping.

---

## 💡 Real-World Examples

### Minimal Setup (Day 1)
```
AGENTS.md                          # What the agent should know
.agent/rules/engineering.md        # How the agent should behave
.agent/skills/review-pr/           # Reusable review process
.agent/templates/pr-description.md # Consistent PR format
```

### Growing Setup (Month 1+)
```
AGENTS.md
.agent/rules/
  ├── engineering.md
  ├── testing.md
  ├── security.md
  └── infrastructure.md
.agent/skills/
  ├── review-pr/
  ├── implement-feature/
  ├── debug-failing-test/
  └── write-adr/
.agent/agents/
  ├── security-reviewer.md
  ├── test-engineer.md
  └── repo-architect.md
.agent/hooks/
  ├── block-dangerous-commands.sh
  └── validate-secret-access.sh
.mcp.json                          # GitHub, Jira, Sentry, etc.
```

---

## ❌ Mistakes to Avoid

1. **Dumping everything into the metadata file** → Split into skills, rules, resources, templates
2. **Using skills for safety enforcement** → Use hooks, permissions, and rules instead
3. **Creating subagents too early** → Start with 1–3, add only after workflow proves useful
4. **Giving tools overly broad permissions** → Separate `run_tests`, `run_lint`, `read_logs`
5. **Confusing rules with skills** → Rules say "how to behave," skills say "how to do tasks"

See [Terminology.md](./Terminology.md#26-common-mistakes) section 26 for full details.

---

## 🎯 Who This Is For

- **Individual developers** starting a new AI-agent-ready project
- **Team leads** designing AI-agent-friendly workflows for your team
- **DevOps engineers** setting up shared agent configuration across repos
- **Security teams** implementing guardrails and hooks for agent actions
- **Product teams** migrating to AI-assisted development
- **Enterprise architects** standardizing agent behavior across codebases

---

## 📖 Contributing

Found a mistake? Have a working example from your tool? See a new agent pattern worth documenting?

**Open an issue or PR.** This guide is a living document. Real-world patterns make it better.

---

## 📄 License

This resource is provided as-is for public use. Share, remix, and adapt freely for your projects.

---

## 🚀 Next Steps

1. **Pick your tool** — Claude Code? Codex? IBM Bob? Cursor? Internal agent?
2. **Read the intro** — Skim [Terminology.md](./Terminology.md) sections 1–3 (taxonomy, metadata, rules)
3. **Find your tool's guide** — Read Calude-Style.md / Codex-Style.md / IBMBob-Style.md
4. **Start minimal** — Create AGENTS.md (or CLAUDE.md) and one rule file
5. **Add skills as you need them** — Don't over-engineer on day one

**Questions?** Each file has examples and decision trees. Use the table of contents to jump to what you need.

Happy building! 🚀
