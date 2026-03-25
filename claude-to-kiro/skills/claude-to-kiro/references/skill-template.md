# Kiro Skill Format (SKILL.md)

## Overview

Kiro adopted the same skill standard as Claude Code. A skill is a `SKILL.md` file
with YAML frontmatter (`name` and `description`) and a markdown body, stored in a
directory that can optionally contain `references/` and `scripts/` subdirectories.

This makes skills the **easiest part of any Claude-to-Kiro migration**. The format
is near-identical — the same frontmatter fields, the same markdown body, the same
directory structure. The only changes needed are tool name translations, file
location adjustments, and removal of any Claude-specific tool references in the
instruction body.

Progressive loading works the same way in both tools: only the `name` and
`description` fields are loaded at startup. The full skill body is loaded on
demand — either when the user explicitly invokes the skill or when the description
matches the user's request (auto-triggering). This keeps startup context lean
regardless of how many skills are installed.

---

## File Locations

### Workspace Skills (Project-Scoped)

```
project-root/
└── .kiro/
    └── skills/
        └── <skill-name>/
            ├── SKILL.md          (required)
            ├── scripts/          (optional — executable logic)
            └── references/       (optional — output format templates)
```

Workspace skills apply only to the current project. This is where converted
Claude Code skills land during migration.

### Global Skills (User-Scoped)

```
~/.kiro/
└── skills/
    └── <skill-name>/
        ├── SKILL.md
        ├── scripts/
        └── references/
```

Global skills apply to every project, equivalent to `~/.claude/skills/<name>/`
in Claude Code. Convert global Claude skills here when running a global config
migration.

---

## Directory Structure

Each skill lives in its own named directory containing at minimum a `SKILL.md`
file. Two optional subdirectories provide supporting content:

| Component | Required | Purpose |
|---|---|---|
| `SKILL.md` | Yes | Skill definition — frontmatter + markdown instructions |
| `scripts/` | No | Executable scripts the skill can reference or invoke |
| `references/` | No | Output format templates, mapping guides, example files |

This is **identical** to Claude Code's skill directory layout. No structural
changes are needed during conversion.

---

## YAML Frontmatter

The frontmatter block is enclosed in `---` fences at the top of `SKILL.md`.
Two fields are required:

```yaml
---
name: skill-name
description: >
  A clear description of what this skill does and when it should be triggered.
  This text is used for auto-triggering — Kiro matches it against the user's
  request to decide whether to load the full skill.
---
```

| Field | Required | Purpose |
|---|---|---|
| `name` | Yes | Unique identifier for the skill |
| `description` | Yes | Used for progressive loading and auto-trigger matching |

**Same as Claude Code.** Both fields serve the same purpose in both tools. The
`description` is particularly important — it controls auto-triggering in both
Claude Code and Kiro. A vague description means the skill won't trigger when
it should; an overly broad description means it triggers when it shouldn't.

---

## Progressive Loading

Both tools use the same progressive loading strategy:

1. **At startup** — only `name` and `description` are read from every installed
   skill. This is fast and keeps the initial context small.
2. **On demand** — the full `SKILL.md` body, `references/`, and `scripts/` are
   loaded when:
   - The user explicitly invokes the skill
   - The `description` matches the user's current request (auto-triggering)

This behaviour is identical in Claude Code and Kiro. No conversion needed.

---

## What Changes During Conversion

### 1. Tool Names

Claude Code and Kiro use different canonical tool names. All references must be
translated in the skill body, references, and scripts:

| Claude Code | Kiro Alias | Kiro Canonical |
|---|---|---|
| `Read` | `read` | `fs_read` |
| `Write` | `write` | `fs_write` |
| `Edit` | `edit` | `fs_edit` |
| `Bash` | `shell` | `shell_execute` |
| `Glob` | `list` | `fs_list` |
| `Grep` | `search` | `fs_search` |
| `WebFetch` | `web_fetch` | `web_fetch` |
| `WebSearch` | `web_search` | `web_search` |

**Convention:** Use the short alias (`read`, `shell`, `search`) in skill prose
for readability. The canonical names (`fs_read`, `shell_execute`, `fs_search`)
are what Kiro uses internally but the aliases are fully supported.

The `Agent` tool (subagent spawning) has no direct tool-name equivalent — it maps
to Kiro custom agents in `.kiro/agents/<name>.md`. If a skill references spawning
subagents, add a `<!-- TODO: Manual migration needed — convert Agent/subagent
references to .kiro/agents/ custom agents -->` comment.

### 2. File Location

Claude Code skills can live anywhere — `skills/<name>/`, `~/.claude/skills/<name>/`,
or within a plugin directory. Kiro skills must live in one of two locations:

| Scope | Kiro Location |
|---|---|
| Workspace | `.kiro/skills/<name>/` |
| Global | `~/.kiro/skills/<name>/` |

### 3. Claude-Specific Tool References in Instructions

Any instruction text that refers to Claude-specific tool behaviour needs updating.
Common patterns:

| Claude Code Reference | Kiro Equivalent |
|---|---|
| "Use the Read tool to..." | "Use the read tool to..." |
| "Run this with Bash..." | "Run this with shell..." |
| "Search with Grep for..." | "Search with search for..." |
| "Use Glob to find files..." | "Use list to find files..." |

---

## What Stays the Same

These elements require **no changes** during conversion:

- **YAML frontmatter format** — same `---` fences, same `name` and `description` fields
- **Markdown body** — same structure, same instruction format (aside from tool names)
- **`references/` subdirectory** — same purpose, same location relative to SKILL.md
- **`scripts/` subdirectory** — same purpose, same location relative to SKILL.md
- **Auto-triggering** — Kiro matches the skill's `description` against the user's
  request, exactly like Claude Code does
- **Progressive loading** — metadata first, full content on demand

---

## Conversion Example

### Source: Claude Code Skill

**`skills/udemy-scanner/SKILL.md`**

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

### Target: Kiro Skill

**`.kiro/skills/udemy-scanner/SKILL.md`**

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

### What Changed

1. **Mapping comment header injected** — documents which tool names were translated,
   placed above the frontmatter so it doesn't interfere with YAML parsing
2. **Tool names in description** — `WebSearch` became `web_search`
3. **Tool names in body** — `WebFetch` became `web_fetch`, `WebSearch` became `web_search`
4. **File location** — moved from `skills/udemy-scanner/` to `.kiro/skills/udemy-scanner/`

### What Stayed the Same

- Frontmatter structure (`name`, `description`)
- Markdown body structure and headings
- Workflow steps and logic
- Any `references/` or `scripts/` directories would be copied as-is (with tool
  names translated in their content)

---

## Mapping Comment Header

Every converted skill that required tool name translation gets a comment header
injected above the frontmatter. This documents the translation for future
maintainers:

```markdown
<!-- Tool names translated from Claude Code -> Kiro:
     Read -> read (fs_read), Write -> write (fs_write), Edit -> edit (fs_edit)
     Bash -> shell (shell_execute), Glob -> list (fs_list), Grep -> search (fs_search)
     WebFetch -> web_fetch, WebSearch -> web_search
     Agent (subagent) -> see .kiro/agents/ custom agents -->
```

Only tool names that actually appear in the converted content are listed in the
header. If a skill only uses `WebSearch` and `WebFetch`, the header only lists
those two translations.

---

## Best Practices

- **Keep descriptions specific for auto-triggering** — The `description` field
  controls when the skill auto-triggers. Write it like a search query: specific
  enough to match relevant requests, broad enough not to miss them. Avoid generic
  descriptions like "helps with coding" — prefer "Search for and extract structured
  Udemy course information."

- **Use `scripts/` for executable logic** — If a skill needs to run shell commands,
  parse files, or perform data transformations, put that logic in `scripts/`. This
  keeps the `SKILL.md` focused on instructions and workflow, with scripts handling
  the implementation.

- **Use `references/` for output format templates** — When a skill produces
  structured output (reports, configs, formatted documents), define the output
  format in a reference template. This separates the "how to do it" (SKILL.md)
  from the "what the output looks like" (references/).

- **Translate tool names in all content** — Tool names need updating not just in
  `SKILL.md` but also in any `references/` and `scripts/` files that mention
  Claude Code tool names.

- **Use short aliases in prose** — Write `read`, `shell`, `search` in skill
  instructions rather than `fs_read`, `shell_execute`, `fs_search`. The aliases
  are more readable and both forms are recognised by Kiro.

- **Flag subagent references for manual review** — If a Claude Code skill spawns
  subagents via the `Agent` tool, this cannot be auto-translated to a tool name.
  It maps to Kiro custom agents (`.kiro/agents/`), which require their own
  configuration files. Add a TODO comment and flag in the migration report.
