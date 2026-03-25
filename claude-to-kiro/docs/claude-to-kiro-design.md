# Claude Code to Kiro — Migration Skill

> **Design Document** | Version 0.1.0 | March 2026
>
> A Claude Code skill that converts Claude configurations into equivalent AWS Kiro
> IDE and CLI configurations. Built for Claude Code users exploring Kiro who want to
> bring their existing skills, rules, hooks, and workflows across.

---

## Purpose

Claude Code and Kiro are both AI-assisted coding tools, but they organise customisation
differently. Engineers who have built skills, rules, hooks, and workflows in Claude Code
often want those same guardrails in Kiro — especially when evaluating Kiro for team adoption
or working across both tools.

This skill automates that translation. Point it at your Claude setup and it generates
the Kiro equivalent, with a migration report showing what ported cleanly, what needs
manual adjustment, and what Kiro features you could adopt that Claude Code doesn't have.

---

## Why Kiro?

Kiro is AWS's agentic AI IDE, available as both a standalone IDE (VS Code fork) and a
CLI. It reached general availability in November 2025 and has evolved rapidly. For Claude
Code users, Kiro is interesting because:

**What Kiro adds beyond Claude Code:**

| Feature | What It Does |
|---|---|
| **Spec-driven development** | Structured requirements → design → tasks workflow using EARS notation |
| **Steering inclusion modes** | Context-sensitive rule loading — `always`, `fileMatch`, `manual`, `auto` |
| **IDE hooks** | File-event triggers (save, create, delete) that run agent prompts or shell commands |
| **Autonomous agent** | Runs up to 10 tasks concurrently in sandboxed environments across sessions |
| **Powers** | Bundled partner integrations (MCP + steering + hooks as an installable unit) |
| **Knowledge bases** | Indexed local directories available as agent resources |
| **Agent `toolsSettings`** | Per-tool path and command restrictions (more granular than Claude permissions) |
| **AGENTS.md support** | Cross-tool agent standard, always included in context |
| **Autonomous agent** | Runs up to 10 tasks concurrently in sandboxed environments (out of scope for migration — no Claude Code equivalent to convert from) |

**What ports cleanly (near-1:1):**

| Claude Code | Kiro | Notes |
|---|---|---|
| Skills (SKILL.md) | Agent Skills (.kiro/skills/) | Same standard — YAML frontmatter (`name`, `description`) + markdown body + `references/` and `scripts/` subdirectories |
| Hooks (PreToolUse, PostToolUse, Stop) | CLI hooks | Same event model, different config format |
| MCP servers | MCP servers | Same protocol, different file location |
| Subagents | Custom agents (.kiro/agents/) | Kiro supports parallel custom agents |
| CLAUDE.md | Steering files (.kiro/steering/) | Kiro adds inclusion modes |
| Plugins | Agents + Steering + Skills + MCP | Decomposed into coordinated components |

**What changes:**

| Area | Difference |
|---|---|
| Tool names | `Read` → `read` (fs_read), `Bash` → `shell` (shell_execute), etc. |
| Project rules | Single CLAUDE.md → multiple steering files with inclusion modes |
| Prompt library | No separate prompt system — procedural content lives in skills or manual steering |
| Progressive disclosure | Claude loads skill metadata first, then full content. Kiro does this for skills but not steering. |
| Permissions | Claude uses settings.json allow/deny. Kiro uses per-agent allowedTools and toolsSettings. |

---

## For Users New to Kiro: How the Mental Model Differs

If you've been using Claude Code and are considering Kiro, this section walks through the
key differences in plain language. The goal is to help you draw your own comparisons so
the migration feels familiar rather than foreign.

### Where Your Project Rules Live

In Claude Code, you have one file — `CLAUDE.md` — sitting in your project root. Everything
goes in there: coding standards, architecture rules, workflow instructions, security policies.
Claude reads the whole thing every time you start a conversation.

Kiro takes a different approach. Instead of one big file, you create **multiple smaller
files** in a `.kiro/steering/` directory. Each file focuses on one topic — say, your
TypeScript conventions, or your security policies, or your deployment workflow.

The advantage is that Kiro lets you control **when** each file gets loaded:

- **Always** — loaded every time, just like CLAUDE.md. Use this for your core project
  rules that always apply.
- **File match** — only loaded when you're working with certain files. If you have rules
  that only matter for React components, you can set them to load only when `.tsx` files
  are involved. This keeps your context cleaner.
- **Manual** — only loaded when you explicitly ask for it by typing `/` or `#steering-name`
  in chat. Good for step-by-step workflows you don't need every conversation.
- **Auto** — loaded when Kiro thinks the steering file is relevant to your current request,
  based on a description you write. Similar to how Claude Code skills auto-trigger.

When you migrate, this skill splits your single CLAUDE.md into multiple steering files and
picks the most appropriate loading mode for each one.

### How Memory and Lessons Work

Claude Code has a persistent memory system — files in `~/.claude/` that remember things
across conversations. Many users also keep a `tasks/lessons.md` file to track gotchas
and patterns they've learned the hard way, so Claude doesn't repeat the same mistakes.

Kiro doesn't have a built-in memory system in the same sense. Instead, it achieves the
same outcome through **global steering files**. These live in `~/.kiro/steering/` and
apply to every project you open, just like Claude's global `~/.claude/CLAUDE.md`.

Here's how your Claude Code patterns translate:

| What You Do in Claude Code | How to Do It in Kiro |
|---|---|
| `~/.claude/CLAUDE.md` with global rules | Create files in `~/.kiro/steering/` — each rule set gets its own file |
| `tasks/lessons.md` with project gotchas | Create `.kiro/steering/lessons.md` with `inclusion: always` |
| `~/.claude/tasks/lessons.md` with global lessons | Create `~/.kiro/steering/lessons.md` (global) |
| Claude memory files (`~/.claude/memory/`) | No direct equivalent — put durable rules in global steering instead |
| "Remember to always do X" | Write it in a steering file — Kiro reads it every time |
| `tasks/todo.md` for task tracking | Kiro has **specs** — a built-in `tasks.md` with real-time status tracking |

The key difference: Claude Code separates "memory" (things it remembers about you) from
"rules" (instructions it follows). Kiro combines both into steering files. If you want
Kiro to remember something, you write it in a steering file. If you want it to follow a
rule, same place.

### Security Rules and Safety Guardrails

Rules like "never expose API keys in chat", "never commit .env files", or "always use
parameterised queries" work the same way in both tools — you write them as instructions
and the AI follows them.

In Claude Code, you put these in your CLAUDE.md:

```markdown
## Security
- Never expose API keys, passwords, or secrets in chat output
- Never commit .env files or credentials
- Always use parameterised queries — never concatenate SQL strings
```

In Kiro, you'd create a `.kiro/steering/security.md` file:

```markdown
---
inclusion: always
---

# Security Policies

- Never expose API keys, passwords, or secrets in chat output
- Never commit .env files or credentials
- Always use parameterised queries — never concatenate SQL strings
```

The content is identical — you're just adding a small header that tells Kiro to always
load this file. The migration skill does this conversion automatically.

Where Kiro goes further than Claude Code on security:

- **Per-agent tool restrictions** — In Claude Code, you set permissions globally in
  `settings.json` (allow these tools, deny those). In Kiro, you can set restrictions
  **per agent**. Your "code review" agent might only be allowed to read files, while
  your "deployment" agent can run shell commands but only specific ones. This is done
  through `allowedTools` and `toolsSettings` in agent configs.

- **Command-level restrictions** — Kiro agents can specify exactly which shell commands
  are allowed or denied. Instead of blanket "allow Bash" or "deny Bash", you can say
  "allow `npm test` and `npm run build` but deny `rm -rf` and `chmod`." Claude Code's
  hook system can achieve similar results but requires more setup.

- **MCP Registry Governance** (enterprise feature) — Administrators can control which
  MCP servers are approved for use across the organisation. Claude Code has no equivalent
  — any user can configure any MCP server.

- **Model Governance** (enterprise feature) — Administrators can control which AI models
  agents are allowed to use. Useful for compliance environments where only certain models
  are approved.

When migrating, the skill converts your Claude Code permission blocks into Kiro's
per-agent restrictions and flags any security rules that could be strengthened using
Kiro's more granular controls. These appear in the "Enhancement Opportunities" section
of the migration report.

### Skills: Almost Identical

This is the easiest part of the migration. Kiro adopted the same skill format as Claude
Code — a `SKILL.md` file with YAML frontmatter (`name` and `description`) and a markdown
body, with optional `references/` and `scripts/` subdirectories.

If you've built skills for Claude Code, they'll work in Kiro with minimal changes:

1. **Tool names** need updating (`Read` becomes `read`, `Bash` becomes `shell`, etc.)
2. **File location** changes from wherever you stored them to `.kiro/skills/<name>/`
3. **Auto-triggering** works the same way — Kiro matches your skill's `description`
   against the user's request, just like Claude Code does

The migration skill handles all three automatically.

### Hooks: Same Events, Different Format

Claude Code hooks fire on events like `PreToolUse` (before a tool runs), `PostToolUse`
(after a tool runs), and `Stop` (when the agent finishes). You use them to block dangerous
commands, validate output, or trigger follow-up actions.

Kiro has the same event model with the same event names (just in camelCase: `preToolUse`,
`postToolUse`, `stop`). The difference is where they're configured:

- **Claude Code** — hooks are defined in `settings.json` or plugin hook files
- **Kiro CLI** — hooks are defined inside agent configuration files, scoped to that agent
- **Kiro IDE** — hooks can also be created through the UI for file-event triggers (like
  "when a file is saved, run the linter"), which is something Claude Code can't do

The migration skill converts your Claude Code hooks into Kiro CLI hook format, translating
event names and tool matchers automatically.

### Subagents: Now Full Custom Agents

In Claude Code, you spawn subagents to handle parallel tasks — research, code review,
testing — each with its own context window. They're ephemeral and defined inline.

Kiro takes this further with **custom agents** in `.kiro/agents/`. These are persistent,
named agents with their own:

- System prompt (what the agent does)
- Tool access (which tools it can use)
- MCP servers (which external services it connects to)
- Resources (which files, skills, and knowledge bases it can reference)
- Hooks (lifecycle events specific to this agent)
- Keyboard shortcut (quick-switch in the IDE)

When migrating, your Claude Code subagent definitions become Kiro custom agent files.
The migration skill generates the full agent config and flags Kiro-only features you
could add (like knowledge bases or per-tool path restrictions) in the enhancement
opportunities section.

---

## The Problem

Claude Code uses a **unified model**: a single `SKILL.md` contains declarative rules,
procedural workflows, tool configs, and examples in one place. `CLAUDE.md` is a single
file for all project context.

Kiro uses a **distributed model**: the same information maps to multiple config locations
with different formats and loading behaviours:

| Content Type | Kiro Location | Loading |
|---|---|---|
| Project rules and conventions | `.kiro/steering/*.md` | Per inclusion mode |
| Reusable skills | `.kiro/skills/<name>/SKILL.md` | Progressive (metadata → full) |
| Agent definitions | `.kiro/agents/<name>.md` | On agent activation |
| Hook logic | Agent config `hooks` block | On matching events |
| MCP tool integrations | `.kiro/settings/mcp.json` | At startup |

Unlike the Amazon Q Developer migration (which required splitting content across rules,
prompts, agents, and MCP), the Kiro migration is substantially more direct — most Claude
Code primitives have near-exact Kiro equivalents. The main work is reformatting configs,
translating tool names, and choosing appropriate steering inclusion modes.

---

## How It Works

### Content Classification

The skill reads each Claude source file and routes content to Kiro destinations:

```
+---------------------------------+
|        Claude Source File        |
|  (CLAUDE.md / SKILL.md / etc.)  |
+----------------+----------------+
                 |
          +------+------+
          |  Classifier  |
          +------+------+
                 |
    +--------+---+---+--------+-----------+
    |        |       |        |           |
    v        v       v        v           v
 Steering  Skills  Agents   Hooks       Report
 (.kiro/   (.kiro/ (.kiro/  (agent      (MIGRATION_
  steering) skills) agents)  config)     REPORT.md)
```

**Routing table:**

| Claude Code Source | Kiro Destination | Steering Inclusion |
|---|---|---|
| CLAUDE.md — core rules, conventions | `.kiro/steering/<topic>.md` | `always` |
| CLAUDE.md — file-type-specific rules | `.kiro/steering/<topic>.md` | `fileMatch` |
| CLAUDE.md — procedural workflows | `.kiro/steering/<topic>.md` | `manual` |
| Skills (SKILL.md + references/) | `.kiro/skills/<name>/SKILL.md` | N/A |
| Hooks (PreToolUse, PostToolUse, Stop) | Agent config `hooks` block | N/A |
| MCP servers (settings.json) | `.kiro/settings/mcp.json` | N/A |
| Plugins (plugin.json + components) | Agents + steering + skills + MCP | Mixed |
| Subagent definitions | `.kiro/agents/<name>.md` | N/A |
| Slash commands | `.kiro/steering/<name>.md` | `manual` |
| CLAUDE.md — AGENTS.md-compatible directives | `AGENTS.md` (project root) | N/A (always included) |

### Steering Inclusion Mode Classification (Hybrid)

The skill uses a hybrid approach: signal-based detection for obvious cases, `always` as
the default, and flags in the migration report for ambiguous content.

**`fileMatch`** — detected when content contains:
- Explicit file patterns: `*.tsx`, `*.py`, `.go files`
- Path references: "in src/components", "under tests/"
- Language scoping: "For TypeScript", "When writing Python", "React components"

**`manual`** — detected when content contains:
- Numbered step workflows, decision trees
- "First... then..." procedural patterns
- Slash command definitions
- Output format templates

**`auto`** — detected when content contains:
- Description-based matching phrases (similar to Claude skill triggering)
- Context-dependent rules that should only load when the request matches a description
- Content that was a standalone Claude skill but is being converted to steering rather
  than a Kiro skill (e.g., when the content is too small for a full skill directory)
- Requires `name` and `description` fields in the steering frontmatter

**`always`** (default) — everything else:
- Architecture rules, coding standards, conventions
- Project structure, dependency notes
- Team constraints, security policies

**Ambiguous** — flagged in migration report when:
- Content mixes declarative and procedural
- File-type scoping is implied but not explicit
- Content could reasonably be `always` or `fileMatch`

### Tool Name Translation

Claude Code and Kiro use different canonical tool names. The skill auto-translates all
references and injects a mapping comment header in every affected file:

| Claude Code | Kiro Canonical | Kiro Alias |
|---|---|---|
| `Read` | `fs_read` | `read` |
| `Write` | `fs_write` | `write` |
| `Edit` | `fs_edit` | `edit` |
| `Bash` | `shell_execute` | `shell` |
| `Glob` | `fs_list` | `list` |
| `Grep` | `fs_search` | `search` |
| `WebFetch` | `web_fetch` | — |
| `WebSearch` | `web_search` | — |
| `Agent` (subagent) | `.kiro/agents/<name>.md` custom agent | Direct mapping (not a tool rename) |

Generated comment header:

```markdown
<!-- Tool names translated from Claude Code -> Kiro:
     Read -> read (fs_read), Write -> write (fs_write), Edit -> edit (fs_edit)
     Bash -> shell (shell_execute), Glob -> list (fs_list), Grep -> search (fs_search)
     WebFetch -> web_fetch, WebSearch -> web_search
     Agent (subagent) -> see .kiro/agents/ custom agents -->
```

---

## Conversion Workflows

### Workflow 1: Single Skill Conversion

Convert one Claude Code skill to a Kiro skill.

```
Source: skills/<name>/SKILL.md + references/ + scripts/
Target: .kiro/skills/<name>/SKILL.md + references/ + scripts/
```

**Steps:**
1. Read source `SKILL.md` and all `references/`, `scripts/`, `examples/`
2. Translate tool names in all content (with mapping comment header)
3. Adjust YAML frontmatter — keep `name` and `description`, drop Claude-specific fields
4. Copy `references/` and `scripts/` directories (tool names translated in content)
5. Generate migration report entry

This is the simplest workflow. Kiro adopted the same SKILL.md standard, so skills
are nearly copy-paste with tool name adjustments.

### Workflow 2: Full Project Conversion

Convert an entire Claude Code project setup to Kiro.

```
Source: CLAUDE.md + skills/ + hooks + MCP + plugins
Target: .kiro/steering/ + .kiro/skills/ + .kiro/agents/ + .kiro/settings/mcp.json
```

**Steps:**
1. Read `CLAUDE.md` → split into steering files by topic, classify inclusion modes
2. Scan `skills/` directory → convert each skill via Workflow 1
3. Read hook configurations → translate to Kiro CLI hook format in agent configs
4. Read MCP config from Claude settings → translate to `.kiro/settings/mcp.json`
5. Read any plugin definitions → convert via Workflow 4
6. Consolidate migration report + enhancement opportunities

### Workflow 3: Global Config Conversion

Convert the user's global Claude Code configuration to Kiro.

```
Source: ~/.claude/CLAUDE.md + ~/.claude/skills/* + ~/.claude/settings.json
Target: ~/.kiro/steering/ + ~/.kiro/skills/ + ~/.kiro/settings/mcp.json
```

**Steps:**
1. Read `~/.claude/CLAUDE.md` → generate global steering files
2. Scan `~/.claude/skills/*` → convert each skill to `~/.kiro/skills/`
3. Read global MCP config → translate to `~/.kiro/settings/mcp.json`
4. Generate migration report

### Workflow 4: Plugin-to-Power Bundle

Convert a Claude Code plugin into a coordinated Kiro bundle of agents, steering,
hooks, and MCP. This is the most complex workflow and the one that most benefits
from automation.

```
Source: plugin.json + skills/ + agents/ + hooks/ + .mcp.json
Target: .kiro/agents/<name>.md + .kiro/steering/ + .kiro/skills/ + .kiro/settings/mcp.json
```

**Steps:**
1. Read `plugin.json` manifest — extract name, description, component list
2. For each skill in the plugin → convert via Workflow 1
3. For each agent definition → generate `.kiro/agents/<name>.md` with:
   - `prompt` from agent system prompt
   - `tools` mapped from Claude tool permissions
   - `allowedTools` from auto-approved tools
   - `hooks` from Claude hook definitions (translated event names and matchers)
   - `mcpServers` from plugin MCP config
   - `resources` pointing to converted steering files and skills
4. For hook-only definitions (no parent agent) → attach to a generated agent or document in steering
5. Merge MCP servers into `.kiro/settings/mcp.json`
6. Generate migration report entry with enhancement opportunities noting Kiro-only features

### Output Modes

| Mode | Behaviour |
|---|---|
| **Staging** (default) | Write all output to `kiro-export/` directory for review |
| **Direct** (explicit only) | Write to `.kiro/` and `~/.kiro/` in-place |

---

## Usage

### Port a Single Skill

> "Port my udemy-scanner skill to Kiro"

Reads `skills/udemy-scanner/SKILL.md` + references and generates:

```
kiro-export/
├── .kiro/
│   └── skills/
│       └── udemy-scanner/
│           ├── SKILL.md
│           └── references/
│               └── (converted reference files)
└── MIGRATION_REPORT.md
```

### Port an Entire Project

> "Convert this project's Claude setup to Kiro"

Reads CLAUDE.md, all skills, hooks, and MCP config. Generates the full Kiro structure:

```
kiro-export/
├── .kiro/
│   ├── steering/
│   │   ├── project-context.md          (inclusion: always)
│   │   ├── typescript-standards.md     (inclusion: fileMatch, pattern: **/*.ts)
│   │   └── task-workflow.md            (inclusion: manual)
│   ├── skills/
│   │   ├── udemy-scanner/
│   │   │   └── SKILL.md
│   │   └── claude-to-kiro/
│   │       └── SKILL.md
│   ├── agents/
│   │   └── (if plugins or subagents exist)
│   └── settings/
│       └── mcp.json
└── MIGRATION_REPORT.md
```

### Port Global Claude Config

> "Port my global Claude config to Kiro"

Reads `~/.claude/CLAUDE.md` and `~/.claude/skills/*`. Generates global Kiro equivalents.

### Port a Plugin

> "Convert my network-tools plugin to Kiro"

Reads the plugin manifest and all components. Generates coordinated agents, steering,
skills, and MCP config.

### Direct Mode

> "Port my Claude skills to Kiro directly — write in-place"

Writes directly to `.kiro/` and `~/.kiro/` instead of the staging directory.

---

## Migration Report

Every conversion generates `MIGRATION_REPORT.md` with five sections:

### 1. Successfully Ported

```markdown
- [x] CLAUDE.md → .kiro/steering/project-context.md (always)
- [x] CLAUDE.md (TypeScript rules) → .kiro/steering/typescript.md (fileMatch: **/*.ts)
- [x] skills/udemy-scanner → .kiro/skills/udemy-scanner/
- [x] PreToolUse hook (block rm -rf) → agent hook with shell_execute matcher
- [x] MCP servers (3/3) → .kiro/settings/mcp.json
```

### 2. Partial — Manual Adjustment Needed

```markdown
- [~] Plugin "network-tools" → .kiro/agents/network-tools.md
      Hook logic simplified — review tools denied mapping
      See TODO comments in generated files
```

### 3. Not Portable — Workarounds Suggested

```markdown
- [ ] Claude context window management (progressive disclosure for steering)
      → Workaround: keep steering files small, use fileMatch to reduce loaded context
- [ ] Claude permission modes (settings.json allow/deny)
      → Workaround: use per-agent allowedTools and toolsSettings
```

### 4. File Mapping Table

```markdown
| Claude Source | Kiro Target | Inclusion | Status |
|---|---|---|---|
| CLAUDE.md (core) | .kiro/steering/project-context.md | always | Ported |
| CLAUDE.md (TS rules) | .kiro/steering/typescript.md | fileMatch | Ported |
| skills/udemy-scanner/ | .kiro/skills/udemy-scanner/ | — | Ported |
| ~/.claude/settings.json | .kiro/settings/mcp.json | — | Ported |
```

### 5. Enhancement Opportunities

Kiro features the user could adopt that have no Claude Code equivalent:

```markdown
- **fileMatch steering**: Your TypeScript rules were converted with
  fileMatch: "**/*.{ts,tsx}" — consider adding more file-scoped rules
  for CSS, tests, or infrastructure files
- **Agent knowledge bases**: Your network-tools agent could reference
  indexed documentation via resources: [{ type: "knowledgeBase", path: "docs/" }]
- **toolsSettings path restrictions**: Lock agents to specific directories
  with toolsSettings.allowedPaths instead of broad tool permissions
- **Kiro specs**: Your project has structured task workflows that could be
  formalised as Kiro specs (requirements.md / design.md / tasks.md)
- **IDE hooks**: Consider adding file-save hooks for auto-linting or
  test-running — these complement CLI hooks with no Claude Code equivalent
```

Inline `<!-- TODO: Manual migration needed — [description] -->` comments are injected
in generated files where manual adjustment is required.

---

## Worked Example: Converting udemy-scanner

This walkthrough shows the complete conversion of a real Claude Code skill.

### Input: Claude Code Skill

**`skills/udemy-scanner/SKILL.md`** (excerpt):
```markdown
---
name: udemy-scanner
description: >
  Search for and extract structured Udemy course information using
  multi-strategy fallbacks via Class Central and WebSearch.
---

## Important: Udemy Blocks Direct Fetching
Udemy returns 403 Forbidden on direct WebFetch requests. Never attempt to
fetch udemy.com URLs directly.

## Workflow

### Single Course Lookup
1. Identify the course URL or title
2. Use WebSearch to find Class Central mirror
3. Use WebFetch on Class Central result
4. Extract structured data
5. If instructor unknown, use WebSearch for instructor subdomain
```

### Output: Kiro Skill

**`kiro-export/.kiro/skills/udemy-scanner/SKILL.md`**:
```markdown
<!-- Tool names translated from Claude Code -> Kiro:
     WebFetch -> web_fetch, WebSearch -> web_search -->

---
name: udemy-scanner
description: >
  Search for and extract structured Udemy course information using
  multi-strategy fallbacks via Class Central and web_search.
---

## Important: Udemy Blocks Direct Fetching
Udemy returns 403 Forbidden on direct web_fetch requests. Never attempt to
fetch udemy.com URLs directly.

## Workflow

### Single Course Lookup
1. Identify the course URL or title
2. Use web_search to find Class Central mirror
3. Use web_fetch on Class Central result
4. Extract structured data
5. If instructor unknown, use web_search for instructor subdomain
```

### Output: Migration Report Entry

```markdown
## 1. Successfully Ported
- [x] skills/udemy-scanner/SKILL.md → .kiro/skills/udemy-scanner/SKILL.md
      Tool names translated: WebSearch → web_search, WebFetch → web_fetch

## 5. Enhancement Opportunities
- **Auto-triggering preserved**: Kiro skills use the same description-based
  matching as Claude Code — this skill will auto-trigger on matching requests
- **Consider adding scripts/**: If the search strategies could be scripted,
  Kiro skills support bundled scripts in scripts/
```

---

## Complete Concept Mapping

### Configuration Files

| Claude Code | Location | Kiro | Location |
|---|---|---|---|
| `CLAUDE.md` (project) | Project root | Steering files | `.kiro/steering/*.md` |
| `CLAUDE.md` (global) | `~/.claude/CLAUDE.md` | Global steering | `~/.kiro/steering/*.md` |
| `settings.json` (MCP) | `~/.claude/settings.json` | MCP config | `.kiro/settings/mcp.json` |
| Plugin config | `.claude-plugin/plugin.json` | Agent + steering + MCP | `.kiro/agents/` + `.kiro/steering/` |
| Skills | `skills/<name>/SKILL.md` | Agent skills | `.kiro/skills/<name>/SKILL.md` |
| CLAUDE.md (agent directives) | Project root | AGENTS.md | Project root (always included) |

### Feature Mapping

| Claude Code Feature | Kiro Equivalent | Mapping Quality |
|---|---|---|
| CLAUDE.md instructions | Steering files (with inclusion modes) | **Better** — adds context-sensitive loading |
| Skills (SKILL.md) | Agent Skills (.kiro/skills/) | **Direct** — same standard |
| Skill references/ | Skill references/ | **Direct** — same structure |
| Skill scripts/ | Skill scripts/ | **Direct** — same structure |
| MCP servers | MCP servers (.kiro/settings/mcp.json) | **Direct** — same protocol |
| Hooks (PreToolUse) | CLI hooks (preToolUse) | **Direct** — same event model |
| Hooks (PostToolUse) | CLI hooks (postToolUse) | **Direct** — same event model |
| Hooks (Stop) | CLI hooks (stop) | **Direct** — same event model |
| Subagents | Custom agents (.kiro/agents/) | **Direct** — parallel execution supported |
| Plugins | Agents + Steering + Skills + MCP | **Good** — decomposed into components |
| Slash commands | Manual steering (`inclusion: manual`) | **Similar** — invoked with `/` in chat or `#steering-name` reference |
| Permissions (allow/deny) | Agent allowedTools + toolsSettings | **Better** — more granular |
| Context window management | Progressive skill loading | **Partial** — skills only, not steering |

---

## Skill Architecture

```
claude-to-kiro/
├── docs/
│   └── claude-to-kiro-design.md              <- This document
└── skills/
    └── claude-to-kiro/
        ├── SKILL.md                           <- Core conversion workflow
        └── references/
            ├── mapping-guide.md               <- Complete concept mapping + classification rules
            ├── steering-template.md           <- .kiro/steering/ format + inclusion modes
            ├── agent-template.md              <- .kiro/agents/ markdown config format
            ├── hook-template.md               <- CLI + IDE hook config format
            ├── skill-template.md              <- .kiro/skills/ format (near-1:1 mapping)
            └── mcp-config-template.md         <- .kiro/settings/mcp.json translation
```

---

## Limitations

- **Kiro specs** — This skill does not generate Kiro spec files (requirements.md,
  design.md, tasks.md) from Claude Code content. Specs are authored development
  artifacts, not configuration — generating them from tangentially related content
  would produce low-quality output. The migration report notes specs as an enhancement
  opportunity.
- **Progressive disclosure for steering** — Claude's progressive disclosure
  (metadata then full content) works for Kiro skills but not steering files.
  Steering files with `inclusion: always` are loaded in full every interaction.
  Workaround: keep steering files small and use `fileMatch` to reduce context.
- **Claude permission modes** — Claude's `settings.json` allow/deny blocks have no
  direct Kiro equivalent at the project level. Workaround: use per-agent `allowedTools`
  and `toolsSettings` for similar restrictions.
- **IDE hooks** — Claude Code hooks are CLI-only. Kiro IDE hooks (file-save triggers,
  etc.) have no Claude Code source material to convert from. Noted as an enhancement
  opportunity in the migration report.
- **Powers** — Kiro Powers are partner-provided bundled integrations (e.g., Figma,
  Netlify). User-created plugins cannot become first-class Powers — they convert to
  the equivalent agent + steering + MCP combination instead.
- **Tool name edge cases** — Auto-translation handles standard tool names. Custom MCP
  tool names or tools referenced by non-standard aliases may not be caught. These are
  flagged in the migration report.
- **Kiro Autonomous Agent** — Kiro's autonomous agent (runs up to 10 concurrent tasks
  in sandboxed environments across sessions) has no Claude Code equivalent to convert
  from. This is a Kiro-native capability noted as an enhancement opportunity in the
  migration report.

---

## Relationship to claude-to-amazonq

This skill exists alongside the `claude-to-amazonq` skill in the same repository.
The two skills are independent — they share no code or templates.

**When to use which:**

| Target | Skill | Notes |
|---|---|---|
| Amazon Q Developer | claude-to-amazonq | IDE plugin, `.amazonq/` structure |
| Kiro IDE or CLI | claude-to-kiro | Standalone IDE/CLI, `.kiro/` structure |
| Migrating Amazon Q → Kiro | Neither — use Kiro's built-in migration | `kiro-cli` auto-migrates `.amazonq/` to `.kiro/` |

---

## Future Enhancements

- **Bidirectional sync** — Kiro → Claude Code conversion
- **Diff-based updates** — Re-run conversion and only update changed files
- **Team templates** — Generate Kiro configs from a team-standard Claude setup
- **Validation** — Verify generated Kiro configs match expected schema
- **AGENTS.md generation** — Generate cross-tool AGENTS.md from Claude CLAUDE.md
  for projects using both tools

---

*Last updated: March 2026*
