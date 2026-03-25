# Claude Code to Kiro — Complete Mapping Guide

## Concept Mapping

### Configuration Files

| Claude Code | Location | Kiro | Location |
|---|---|---|---|
| `CLAUDE.md` (project) | Project root | Steering files | `.kiro/steering/*.md` |
| `CLAUDE.md` (global) | `~/.claude/CLAUDE.md` | Global steering | `~/.kiro/steering/*.md` |
| `settings.json` (MCP) | `~/.claude/settings.json` | MCP config | `.kiro/settings/mcp.json` |
| Plugin config | `.claude-plugin/plugin.json` | Agent + steering + MCP | `.kiro/agents/` + `.kiro/steering/` + `.kiro/settings/mcp.json` |
| Skills | `skills/<name>/SKILL.md` | Agent skills | `.kiro/skills/<name>/SKILL.md` |
| CLAUDE.md (agent directives) | Project root | AGENTS.md | Project root `AGENTS.md` (always included) |

### Feature Mapping

| Claude Code Feature | Kiro Equivalent | Mapping Quality |
|---|---|---|
| CLAUDE.md instructions | Steering files (with inclusion modes) | **Better** — adds context-sensitive loading |
| Skills (SKILL.md) | Agent Skills (`.kiro/skills/`) | **Direct** — same standard |
| Skill `references/` | Skill `references/` | **Direct** — same structure |
| Skill `scripts/` | Skill `scripts/` | **Direct** — same structure |
| Skill `examples/` | Inline into skill or steering | **Partial** — no separate examples directory |
| MCP servers | MCP servers (`.kiro/settings/mcp.json`) | **Direct** — same protocol, different file location |
| Hooks (PreToolUse) | CLI hooks (`preToolUse`) | **Direct** — same event model |
| Hooks (PostToolUse) | CLI hooks (`postToolUse`) | **Direct** — same event model |
| Hooks (Stop) | CLI hooks (`stop`) | **Direct** — same event model |
| Subagents | Custom agents (`.kiro/agents/`) | **Direct** — parallel execution supported |
| Plugins | Agents + Steering + Skills + MCP | **Good** — decomposed into coordinated components |
| Slash commands | Manual steering (`inclusion: manual`) | **Similar** — invoked with `/` or `#steering-name` |
| Permissions (allow/deny) | Agent `allowedTools` + `toolsSettings` | **Better** — more granular, per-agent |
| Context window management | Progressive skill loading | **Partial** — skills only, not steering |
| AGENTS.md directives | AGENTS.md (project root) | **Direct** — cross-tool standard, always included |

---

## Content Classification Rules

Route each section of a Claude source file to a Kiro destination based on signal patterns.

### Steering — `.kiro/steering/*.md`

**Signals:**
- Contains directive language: "Always", "Never", "Prefer", "Must", "Do not", "Required"
- Describes: architecture, conventions, file structure, dependencies
- Establishes: standards, constraints, team agreements, coding style
- Pattern: `<imperative verb> + <standard/convention>`

### Skills — `.kiro/skills/<name>/SKILL.md`

**Signals:**
- Has YAML frontmatter with `name` and `description`
- Contains workflow steps tied to tool usage
- References `references/` or `scripts/` subdirectories
- Designed for reuse across projects
- Pattern: complete self-contained capability with triggering description

### Agents — `.kiro/agents/<name>.md`

**Signals:**
- Defines a specialised role or persona (e.g., "code reviewer", "deployment agent")
- Specifies tool permissions or restrictions
- Contains subagent spawn patterns in Claude Code
- Describes parallel or isolated task execution
- Pattern: `<role> that <does X> using <tools Y>`

### Hooks — Agent config `hooks` block

**Signals:**
- References event names: `PreToolUse`, `PostToolUse`, `Stop`
- Contains tool matchers or command filters
- Describes: validation, blocking, or follow-up actions on tool events
- Pattern: `Before/After <tool> runs, <check/validate/block>`

### AGENTS.md — Project root

**Signals:**
- Cross-tool agent directives meant for any AI coding assistant
- Content written in a tool-agnostic style (no Claude-specific or Kiro-specific tool names)
- Team-level conventions that should work regardless of which AI tool is used
- Pattern: generic agent instructions without tool-specific references

### Migration Report — `MIGRATION_REPORT.md`

**Signals:**
- References Claude-specific APIs (subagent spawning internals, hook event internals)
- Depends on Claude's context window behaviour or skill triggering internals
- Uses features with no Kiro equivalent and no reasonable workaround
- Contains Claude-specific tool names that cannot be auto-translated (custom MCP aliases)

---

## Steering Inclusion Mode Classification

When routing content to steering files, classify the inclusion mode using these signals.

### `fileMatch` — File-pattern scoped

**Signals:**
- Explicit file patterns: `*.tsx`, `*.py`, `.go files`, `*.test.ts`
- Path references: "in `src/components`", "under `tests/`", "files in `infrastructure/`"
- Language scoping: "For TypeScript", "When writing Python", "React components"
- Framework scoping: "For Next.js pages", "In Django views"

**Frontmatter example:**
```yaml
---
inclusion: fileMatch
globs:
  - "**/*.ts"
  - "**/*.tsx"
---
```

### `manual` — On-demand invocation

**Signals:**
- Numbered step workflows, decision trees
- "First... then... next... finally" procedural patterns
- Slash command definitions (Claude `/command` → Kiro manual steering)
- Output format templates
- Rarely-used reference procedures

**Frontmatter example:**
```yaml
---
inclusion: manual
---
```

### `auto` — Description-based matching

**Signals:**
- Description-based matching phrases (similar to Claude skill auto-triggering)
- Context-dependent rules that should only load when the request matches a description
- Content that was a standalone Claude skill but is being converted to steering (too small for a full skill directory)
- Requires `name` and `description` fields in the steering frontmatter

**Frontmatter example:**
```yaml
---
inclusion: auto
name: api-design-guidelines
description: Guidelines for designing REST API endpoints and response formats
---
```

### `always` — Default inclusion

**Signals (or absence of other signals):**
- Architecture rules, coding standards, conventions
- Project structure, dependency notes
- Team constraints, security policies
- Content that does not match `fileMatch`, `manual`, or `auto` patterns

**Frontmatter example:**
```yaml
---
inclusion: always
---
```

### Ambiguous — Flag in migration report

**Signals:**
- Content mixes declarative and procedural
- File-type scoping is implied but not explicit
- Content could reasonably be `always` or `fileMatch`
- Default to `always` and add a migration report note for manual review

---

## Tool Name Translation

Claude Code and Kiro use different canonical tool names. The skill auto-translates all
references in converted files.

### Translation Table

| Claude Code | Kiro Canonical | Kiro Alias | Notes |
|---|---|---|---|
| `Read` | `fs_read` | `read` | File reading |
| `Write` | `fs_write` | `write` | File writing |
| `Edit` | `fs_edit` | `edit` | File editing (string replacement) |
| `Bash` | `shell_execute` | `shell` | Shell command execution |
| `Glob` | `fs_list` | `list` | File pattern matching |
| `Grep` | `fs_search` | `search` | Content search (ripgrep-based) |
| `WebFetch` | `web_fetch` | — | HTTP fetch |
| `WebSearch` | `web_search` | — | Web search |
| `Agent` (subagent) | `.kiro/agents/<name>.md` custom agent | — | Not a tool rename — becomes a custom agent definition |

### Tool Name Mapping Comment Header

Inject this comment block at the top of **every** file that contains translated tool names:

```markdown
<!-- Tool names translated from Claude Code -> Kiro:
     Read -> read (fs_read), Write -> write (fs_write), Edit -> edit (fs_edit)
     Bash -> shell (shell_execute), Glob -> list (fs_list), Grep -> search (fs_search)
     WebFetch -> web_fetch, WebSearch -> web_search
     Agent (subagent) -> see .kiro/agents/ custom agents -->
```

**Rules for the comment header:**
- Only include tool mappings that were actually translated in the file
- Place before the YAML frontmatter (if present) or at the top of the file
- If no tool names were translated, do not inject the comment header

---

## Priority Mapping

Claude Code has no explicit priority system. When converting directive language to
Kiro steering files, classify priority based on language strength to inform ordering
and emphasis in the generated content.

| Claude Language | Priority Level | Steering Guidance |
|---|---|---|
| "MUST", "NEVER", "CRITICAL" | **Critical** | Place in `inclusion: always` steering; use bold or uppercase emphasis |
| "Always", "Never", "Required" | **High** | Place in `inclusion: always` steering; keep near top of file |
| "Prefer", "Should", "Recommended" | **Medium** | Can use `always` or `fileMatch` depending on scope |
| "Consider", "May", "Optional" | **Low** | Candidates for `manual` or `auto` inclusion if context-dependent |

---

## Naming Conventions

### Claude Code

- Skills: `kebab-case` directory names (`my-skill/`)
- Skill entry: `SKILL.md` (uppercase)
- Project config: `CLAUDE.md` (uppercase)
- References: `references/` subdirectory
- Scripts: `scripts/` subdirectory

### Kiro

- Skills: `kebab-case` directory names (`.kiro/skills/my-skill/`)
- Skill entry: `SKILL.md` (uppercase — same as Claude)
- Steering files: `kebab-case.md` in `.kiro/steering/`
- Agents: `kebab-case.md` in `.kiro/agents/`
- MCP config: `.kiro/settings/mcp.json`
- References: `references/` subdirectory (same as Claude)
- Scripts: `scripts/` subdirectory (same as Claude)

### Conversion Rules

| Claude Source | Kiro Target | Example |
|---|---|---|
| `SKILL.md` | `SKILL.md` | No change — same standard |
| `CLAUDE.md` (project) | `.kiro/steering/<topic>.md` | `CLAUDE.md` → `project-context.md`, `typescript.md`, etc. |
| `CLAUDE.md` (global) | `~/.kiro/steering/<topic>.md` | Split by topic, `kebab-case` filenames |
| Skill directory | `.kiro/skills/<name>/` | `my-skill/` → `.kiro/skills/my-skill/` |
| Plugin | `.kiro/agents/<name>.md` + components | Decomposed into agent + steering + MCP |
| Subagent definition | `.kiro/agents/<name>.md` | Named after the subagent's role |
| Slash command | `.kiro/steering/<command-name>.md` | With `inclusion: manual` |

---

*Last updated: March 2026*
