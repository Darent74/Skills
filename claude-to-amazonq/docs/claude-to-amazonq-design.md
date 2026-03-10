# Claude Code to Amazon Q Developer — Migration Skill

> **Design Document** | Version 0.1.0 | March 2026
>
> A Claude Code skill that converts Claude configurations into equivalent Amazon Q Developer
> configurations. Built for Cisco engineers working across both platforms in enterprise environments.

---

## Purpose

Claude Code and Amazon Q Developer are both AI-assisted coding tools, but they organise
customisation differently. Engineers who build skills, rules, and workflows in Claude Code
often need those same guardrails in Amazon Q — especially in AWS-heavy enterprise environments.

This skill automates that translation. Instead of manually rewriting configs, point it at
your Claude setup and it generates the Amazon Q equivalent.

---

## The Problem

Claude Code uses a **unified model**: a single `SKILL.md` file contains declarative rules,
procedural workflows, tool configs, and examples in one place.

Amazon Q Developer uses a **component model**: the same information must be split across
multiple systems:

| Content Type | Amazon Q Location |
|---|---|
| Coding standards and conventions | `.amazonq/rules/*.md` |
| Reusable workflows and prompts | `~/.aws/amazonq/prompts/*.md` |
| Tool and API integrations | `.amazonq/default.json` |
| Specialised agent behaviours | Custom agent JSON configs |

Manually splitting a Claude skill across four systems is tedious and error-prone.
This skill handles it automatically.

---

## How It Works

### Content Classification

The skill reads each Claude source file and classifies every content block:

```
┌─────────────────────────────────┐
│        Claude Source File        │
│  (CLAUDE.md / SKILL.md / etc.)  │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │  Classifier  │
        └──────┬──────┘
               │
    ┌──────────┼──────────┬────────────┬──────────────┐
    ▼          ▼          ▼            ▼              ▼
 Rules      Prompts    MCP Config   Agent Config   Report
 (.amazonq/  (~/.aws/   (.amazonq/   (.amazonq/   (MIGRATION_
  rules/)    amazonq/    default.     agents/)      REPORT.md)
             prompts/)   json)
```

**Classification signals:**

| Signal | Routes To |
|---|---|
| "Always", "Never", "Prefer", "Must" | Rules |
| Numbered steps, "First... then..." | Prompts |
| Server URLs, API endpoints, auth tokens | MCP Config |
| Agent definitions, permissions | Agent Config |
| Hooks, subagents, Claude-specific features | Migration Report |

### Output Modes

| Mode | Flag | Behaviour |
|---|---|---|
| **Staging** (default) | — | Writes to `amazonq-export/` for review |
| **Direct** | `--direct` | Writes to `.amazonq/` and `~/.aws/amazonq/` in-place |

---

## Usage

### Port a Single Skill

> "Port my udemy-scanner skill to Amazon Q"

Reads `skills/udemy-scanner/SKILL.md` + references and generates:

```
amazonq-export/
├── .amazonq/rules/udemy-scanner-rules.md
├── prompts/udemy-scanner.md
└── MIGRATION_REPORT.md
```

### Port an Entire Project

> "Convert this project's Claude setup to Amazon Q"

Reads `CLAUDE.md`, all skills, and MCP config. Generates the full Amazon Q structure:

```
amazonq-export/
├── .amazonq/
│   ├── rules/
│   │   ├── project-context.md
│   │   ├── udemy-scanner-rules.md
│   │   └── claude-to-amazonq-rules.md
│   └── default.json
├── prompts/
│   ├── udemy-scanner.md
│   └── claude-to-amazonq.md
├── agents/
│   └── (if plugins exist)
└── MIGRATION_REPORT.md
```

### Port Global Claude Config

> "Port my global Claude config to Amazon Q"

Reads `~/.claude/CLAUDE.md` and `~/.claude/skills/*`. Generates global Amazon Q equivalents.

### Direct Mode

> "Port my Claude skills to Amazon Q directly — write in-place"

Writes directly to `.amazonq/` and `~/.aws/amazonq/prompts/` instead of staging.

---

## Migration Report

Every conversion generates `MIGRATION_REPORT.md` with:

### 1. Successfully Ported
```markdown
- [x] CLAUDE.md → .amazonq/rules/project-context.md
- [x] skills/udemy-scanner/SKILL.md → rules + prompts
```

### 2. Partial — Manual Adjustment Needed
```markdown
- [~] MCP config: 2 of 3 servers ported
      See TODO comments in .amazonq/default.json
```

### 3. Not Portable — Workarounds Suggested
```markdown
- [ ] PreToolUse hooks → Suggested: use custom agent tools.denied
- [ ] Subagent orchestration → Suggested: create multiple specialised agents
- [ ] Skill auto-triggering → Suggested: document @prompt-name invocation
```

### 4. File Mapping Table
```markdown
| Claude Source | Amazon Q Target | Status |
|---|---|---|
| CLAUDE.md | .amazonq/rules/project-context.md | Ported |
| skills/udemy-scanner/SKILL.md | rules/ + prompts/ | Ported |
| settings.json (MCP) | .amazonq/default.json | Partial |
```

Inline `<!-- TODO: Manual migration needed -->` comments are injected in generated
files where manual adjustment is required.

---

## Concept Mapping Reference

### Configuration Files

| Claude Code | Amazon Q Developer |
|---|---|
| `CLAUDE.md` (project) | `.amazonq/rules/*.md` |
| `~/.claude/CLAUDE.md` (global) | `~/.aws/amazonq/prompts/*.md` |
| `settings.json` (MCP) | `.amazonq/default.json` |
| `.claude-plugin/plugin.json` | Custom agent JSON |

### Features

| Claude Feature | Amazon Q Equivalent | Quality |
|---|---|---|
| CLAUDE.md instructions | Rules (auto-loaded) | Direct |
| Skills (SKILL.md) | Rules + Prompts (split) | Good — requires classification |
| MCP servers | MCP servers | Direct — same protocol, different schema |
| Plugins | Custom agents | Partial |
| Hooks | No equivalent | Workarounds documented |
| Subagents | No equivalent | Workarounds documented |
| Slash commands | `@prompt-name` | Similar invocation |

---

## Skill Architecture

```
skills/claude-to-amazonq/
├── SKILL.md                              ← Core conversion workflow
└── references/
    ├── mapping-guide.md                  ← Complete concept mapping
    ├── rules-template.md                 ← .amazonq/rules/ format + examples
    ├── prompt-template.md                ← Prompt library format + examples
    ├── custom-agent-template.md          ← Custom agent JSON schema + examples
    └── mcp-config-template.md            ← MCP config translation guide
```

---

## Limitations

- **Hooks** — Claude's PreToolUse/PostToolUse/Stop hooks have no Amazon Q equivalent.
  Partial workaround via custom agent `tools.denied` for restriction-type hooks.
- **Subagents** — Claude can spawn parallel subagents. Amazon Q agents are single-threaded.
  Workaround: create multiple specialised agents.
- **Skill auto-triggering** — Claude skills trigger automatically based on description matching.
  Amazon Q prompts must be explicitly invoked with `@prompt-name`.
- **Context window management** — Claude's progressive disclosure (metadata → SKILL.md → references)
  has no Amazon Q parallel. All rules are loaded simultaneously.
- **Scripts** — Claude skills can bundle executable scripts. Amazon Q has no equivalent.
  Scripts must be committed to the repo and referenced in documentation.

---

## Future Enhancements

- **Bidirectional sync** — Amazon Q → Claude Code conversion
- **Diff-based updates** — Re-run conversion and only update changed files
- **Team templates** — Generate Amazon Q configs from a team-standard Claude setup
- **Validation script** — Verify generated Amazon Q configs are syntactically valid

---

*Last updated: March 2026*
