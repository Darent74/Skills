# Claude-to-Kiro Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the complete claude-to-kiro skill — SKILL.md and six reference templates — so it can convert Claude Code configurations into equivalent Kiro IDE/CLI configurations.

**Architecture:** Documentation-only skill following the repo's existing convention: one SKILL.md containing the core conversion workflow, six reference templates in `references/` covering each Kiro output format. No executable code — the skill guides Claude's behaviour through structured markdown. Mirrors the structure of the sibling `claude-to-amazonq` skill.

**Tech Stack:** Markdown only. No build system, no tests, no dependencies.

**Spec:** `claude-to-kiro/docs/claude-to-kiro-design.md`

**Reference for structure:** `claude-to-amazonq/skills/claude-to-amazonq/` (sibling skill to mirror)

---

## File Map

| Action | File | Responsibility |
|---|---|---|
| Create | `claude-to-kiro/skills/claude-to-kiro/SKILL.md` | Core conversion workflow — routing, classification, four workflows, output modes, migration report |
| Create | `claude-to-kiro/skills/claude-to-kiro/references/mapping-guide.md` | Complete Claude → Kiro concept mapping, classification signals, tool name table, naming conventions |
| Create | `claude-to-kiro/skills/claude-to-kiro/references/steering-template.md` | `.kiro/steering/` format spec — frontmatter, inclusion modes, conversion examples |
| Create | `claude-to-kiro/skills/claude-to-kiro/references/agent-template.md` | `.kiro/agents/` markdown config format — all fields, conversion from Claude plugins/subagents |
| Create | `claude-to-kiro/skills/claude-to-kiro/references/hook-template.md` | Kiro CLI + IDE hook format — event names, matchers, conversion from Claude hooks |
| Create | `claude-to-kiro/skills/claude-to-kiro/references/skill-template.md` | `.kiro/skills/` format — near-1:1 mapping, what changes (tool names, location, frontmatter) |
| Create | `claude-to-kiro/skills/claude-to-kiro/references/mcp-config-template.md` | `.kiro/settings/mcp.json` translation — schema comparison, path mapping, conversion example |
| Modify | `INDEX.md` | Add claude-to-kiro entry with description and usage examples |

---

## Task 1: SKILL.md — Core Conversion Workflow

The SKILL.md is the primary file Claude reads when the skill is invoked. It defines the conversion workflow, content classification, and output structure. Mirror the structure of `claude-to-amazonq/skills/claude-to-amazonq/SKILL.md` but with Kiro targets.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/SKILL.md`
- Reference: `claude-to-amazonq/skills/claude-to-amazonq/SKILL.md` (structural template)
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (design spec — sections "Content Classification", "Conversion Workflows", "Migration Report")

**Content requirements:**
- YAML frontmatter: `name: claude-to-kiro`, `description` with trigger phrases (port to Kiro, convert skills to Kiro, migrate config to Kiro, export to Kiro, translate CLAUDE.md to steering, convert hooks to Kiro, convert plugin to Kiro), `version: 0.1.0`
- Structural differences table: Claude Code → Kiro mapping (CLAUDE.md → steering, skills → skills, hooks → hooks, MCP → MCP, plugins → agents+steering+skills+MCP, subagents → custom agents)
- Content classification section with six routing destinations (steering, skills, agents, hooks, AGENTS.md, migration report)
- AGENTS.md routing: CLAUDE.md agent directives → `AGENTS.md` in project root (always included by Kiro)
- Slash command routing: Claude slash commands → `.kiro/steering/<name>.md` with `inclusion: manual`
- Steering inclusion mode signals (fileMatch, manual, auto, always default)
- Ambiguous content handling: flag in migration report when content could map to multiple inclusion modes
- Four conversion workflows (single skill, full project, global config, plugin-to-power) — reference the design doc steps
- Output modes (staging to `kiro-export/`, direct to `.kiro/`)
- Reference callouts: "Consult `references/<name>.md` for..." for each of the six templates
- Migration report structure: five sections (Successfully Ported, Partial, Not Portable, File Mapping Table, Enhancement Opportunities)

- [ ] **Step 1: Read the amazonq SKILL.md for structural reference**

Read `claude-to-amazonq/skills/claude-to-amazonq/SKILL.md` to understand the section ordering and formatting conventions used in the sibling skill.

- [ ] **Step 2: Read the design doc sections needed for SKILL.md content**

Read `claude-to-kiro/docs/claude-to-kiro-design.md` — specifically the "Content Classification", "Conversion Workflows", "Migration Report", and "Complete Concept Mapping" sections.

- [ ] **Step 3: Write SKILL.md**

Create the file at `claude-to-kiro/skills/claude-to-kiro/SKILL.md`. Follow the amazonq SKILL.md structure but with Kiro targets. Include all content from the design spec's routing table, classification signals, four workflows, output modes, and reference callouts.

Key differences from the amazonq SKILL.md:
- Routing destinations are steering/skills/agents/hooks/report (not rules/prompts/agents/MCP/report)
- Add steering inclusion mode classification section
- Four workflows instead of three (add plugin-to-power)
- Migration report has five sections instead of four (add Enhancement Opportunities)
- Reference callouts point to six templates instead of five

- [ ] **Step 4: Verify SKILL.md completeness**

Check that SKILL.md:
- Has valid YAML frontmatter with name, description, version
- References all six template files in `references/`
- Covers all four conversion workflows
- Documents the five-section migration report format
- Includes the steering inclusion mode classification

- [ ] **Step 5: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/SKILL.md
git commit -m "Add claude-to-kiro SKILL.md with core conversion workflow

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 2: mapping-guide.md — Complete Concept Mapping

The mapping guide is the most comprehensive reference file. It contains the full Claude → Kiro concept mapping, classification rules, tool name translation table, priority mapping, and naming conventions.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/references/mapping-guide.md`
- Reference: `claude-to-amazonq/skills/claude-to-amazonq/references/mapping-guide.md` (structural template)
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (sections "Complete Concept Mapping", "Tool Name Translation", "Content Classification")

**Content requirements:**
- Configuration files mapping table (CLAUDE.md → steering, global CLAUDE.md → global steering, settings.json → mcp.json, plugin.json → agents+steering, skills → skills, AGENTS.md → AGENTS.md)
- Feature mapping table with mapping quality ratings (Direct, Better, Good, Similar, Partial)
- Content classification rules with signal patterns for each routing destination
- Steering inclusion mode classification signals (fileMatch, manual, auto, always)
- Tool name translation table (Read → read/fs_read, Write → write/fs_write, Edit → edit/fs_edit, Bash → shell/shell_execute, Glob → list/fs_list, Grep → search/fs_search, WebFetch → web_fetch, WebSearch → web_search, Agent → .kiro/agents/)
- Tool name mapping comment header template
- Priority mapping (MUST/NEVER/CRITICAL → Critical, Always/Never → High, Prefer/Should → Medium, Consider/May → Low) — used when converting CLAUDE.md rules to steering files
- Naming conventions (Claude kebab-case → Kiro kebab-case, SKILL.md → SKILL.md, CLAUDE.md → steering files)

- [ ] **Step 1: Read the amazonq mapping guide for structural reference**

Read `claude-to-amazonq/skills/claude-to-amazonq/references/mapping-guide.md`.

- [ ] **Step 2: Write mapping-guide.md**

Create `claude-to-kiro/skills/claude-to-kiro/references/mapping-guide.md` following the amazonq guide's section ordering but with Kiro-specific content. Include all tables and classification rules from the design spec.

- [ ] **Step 3: Verify mapping-guide.md completeness**

Check that the guide includes: config file mapping, feature mapping with quality ratings, classification rules for all five destinations, steering inclusion mode signals, full tool name table with comment header template, priority mapping, naming conventions.

- [ ] **Step 4: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/references/mapping-guide.md
git commit -m "Add mapping guide with Claude-to-Kiro concept mapping

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 3: steering-template.md — Kiro Steering File Format

This template documents the `.kiro/steering/` file format — the primary destination for CLAUDE.md content. It's the Kiro equivalent of the amazonq `rules-template.md` but richer because of inclusion modes.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/references/steering-template.md`
- Reference: `claude-to-amazonq/skills/claude-to-amazonq/references/rules-template.md` (structural template)
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (sections "Steering Inclusion Mode Classification", "For Users New to Kiro: Where Your Project Rules Live")

**Content requirements:**
- Overview: what steering files are and how they differ from CLAUDE.md
- File locations: `.kiro/steering/` (workspace) and `~/.kiro/steering/` (global), workspace takes precedence
- YAML frontmatter format with all four inclusion modes:
  - `inclusion: always` (default) — loaded every interaction
  - `inclusion: fileMatch` + `fileMatchPattern: "**/*.tsx"` — loaded when working with matching files
  - `inclusion: manual` — loaded on-demand via `/` or `#steering-name`
  - `inclusion: auto` + `name` + `description` — loaded when request matches description
- Foundational templates Kiro can generate: `product.md`, `tech.md`, `structure.md`
- File references syntax: `#[[file:path/to/file]]`
- Conversion example 1: CLAUDE.md core rules → steering with `inclusion: always`
- Conversion example 2: CLAUDE.md file-type rules → steering with `inclusion: fileMatch`
- Conversion example 3: CLAUDE.md workflow → steering with `inclusion: manual`
- Conversion example 4: Claude skill too small for full skill → steering with `inclusion: auto`
- Best practices: one concern per file, keep under 2000 words, use fileMatch to reduce context, set appropriate inclusion mode

- [ ] **Step 1: Read the amazonq rules template for structural reference**

Read `claude-to-amazonq/skills/claude-to-amazonq/references/rules-template.md`.

- [ ] **Step 2: Write steering-template.md**

Create `claude-to-kiro/skills/claude-to-kiro/references/steering-template.md`. Follow the amazonq template's structure (overview, file location, format, examples, best practices) but with steering-specific content. Include all four inclusion modes with frontmatter examples and four conversion examples showing different modes.

- [ ] **Step 3: Verify steering-template.md completeness**

Check: all four inclusion modes documented with frontmatter syntax, both workspace and global locations, at least four conversion examples (one per mode), best practices section, file reference syntax.

- [ ] **Step 4: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/references/steering-template.md
git commit -m "Add steering template with inclusion modes and conversion examples

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 4: agent-template.md — Kiro Custom Agent Format

This template documents the `.kiro/agents/` markdown config format — the destination for Claude plugins and subagent definitions.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/references/agent-template.md`
- Reference: `claude-to-amazonq/skills/claude-to-amazonq/references/custom-agent-template.md` (structural template)
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (sections "Workflow 4: Plugin-to-Power Bundle", "Subagents: Now Full Custom Agents")

**Content requirements:**
- Overview: what Kiro custom agents are and how they differ from Claude plugins/subagents
- File locations: `.kiro/agents/` (workspace) and `~/.kiro/agents/` (global)
- Full configuration reference with all fields:
  - `name`, `description` — agent identity
  - `prompt` — system prompt (inline string or `file://` URI)
  - `model` — model ID (e.g., `claude-sonnet-4`)
  - `tools` — array: `"read"`, `"write"`, `"shell"`, `"aws"`, `"@server_name"`, `"*"`
  - `allowedTools` — tools that run without permission prompts (glob support)
  - `toolsSettings` — per-tool config: `allowedPaths`, `allowedCommands`, `deniedCommands`
  - `mcpServers` — MCP servers available to this agent
  - `includeMcpJson` — boolean: inherit workspace/global MCP configs
  - `resources` — array of `file://`, `skill://`, or `knowledgeBase` objects
  - `hooks` — object with trigger arrays (`agentSpawn`, `preToolUse`, `postToolUse`, `stop`)
  - `toolAliases` — remap tool names
  - `keyboardShortcut` — quick-switch shortcut (IDE only)
  - `welcomeMessage` — displayed on agent activation
- Conversion example 1: Claude plugin → Kiro agent (plugin.json → agent markdown)
- Conversion example 2: Claude subagent definition → Kiro custom agent
- What cannot be converted: ephemeral subagent spawning patterns, inline agent definitions
- Best practices: one domain per agent, use resources for context, restrict tools, name clearly

- [ ] **Step 1: Read the amazonq custom agent template for structural reference**

Read `claude-to-amazonq/skills/claude-to-amazonq/references/custom-agent-template.md`.

- [ ] **Step 2: Write agent-template.md**

Create `claude-to-kiro/skills/claude-to-kiro/references/agent-template.md`. Include the full configuration reference, two conversion examples, and best practices.

- [ ] **Step 3: Verify agent-template.md completeness**

Check: all 14 agent config fields documented (`name`, `description`, `prompt`, `model`, `tools`, `allowedTools`, `toolsSettings`, `mcpServers`, `includeMcpJson`, `resources`, `hooks`, `toolAliases`, `keyboardShortcut`, `welcomeMessage`), file locations for workspace and global, two conversion examples, non-portable items listed, best practices section.

- [ ] **Step 4: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/references/agent-template.md
git commit -m "Add agent template with Kiro custom agent config reference

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 5: hook-template.md — Kiro Hook Format

This template is entirely new — the amazonq skill has no equivalent because hooks were "not portable." For Kiro, hooks are a direct mapping.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/references/hook-template.md`
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (sections "Hooks: Same Events, Different Format", "Tool Name Translation")

**Content requirements:**
- Overview: what Kiro hooks are and the two types (CLI agent hooks, IDE file-event hooks)
- CLI hook format:
  - Event types: `agentSpawn`, `userPromptSubmit`, `preToolUse`, `postToolUse`, `stop`
  - Configuration within agent files (hooks object with event arrays)
  - Matcher field: canonical names (`fs_read`), aliases (`read`), MCP patterns (`@git/status`), wildcards (`*`)
  - Hook receives JSON via STDIN: event name, cwd, tool details
  - Exit codes: 0 = success, 2 = block tool (preToolUse only)
  - `timeout_ms` (default 30s) and `cache_ttl_seconds`
- IDE hook format:
  - Triggers: file saved, created, deleted; user prompt submit; agent stop; manual trigger
  - Actions: "Ask Kiro" (agent prompt) or "Run Command" (shell command)
  - File pattern matching with glob patterns
  - Configured via UI or natural language
- Event name translation table: Claude `PreToolUse` → Kiro `preToolUse`, `PostToolUse` → `postToolUse`, `Stop` → `stop`
- Tool matcher translation: Claude `Bash` → Kiro `shell_execute` or `shell`, etc.
- Conversion example 1: Claude PreToolUse hook blocking `rm -rf` → Kiro CLI preToolUse hook
- Conversion example 2: Claude PostToolUse hook → Kiro CLI postToolUse hook
- What cannot be converted: IDE file-event hooks (no Claude source), SessionStart/SessionEnd hooks (different Kiro event model)
- Best practices: scope hooks to agents, use specific matchers over wildcards, keep timeouts reasonable

- [ ] **Step 1: Write hook-template.md**

Create `claude-to-kiro/skills/claude-to-kiro/references/hook-template.md`. This has no amazonq equivalent to reference — write from the design spec and Kiro documentation.

- [ ] **Step 2: Verify hook-template.md completeness**

Check: both CLI and IDE hook types documented, all five CLI event types enumerated (`agentSpawn`, `userPromptSubmit`, `preToolUse`, `postToolUse`, `stop`), event name translation table (Claude PascalCase → Kiro camelCase), tool matcher translation, two conversion examples, non-portable items listed, best practices.

- [ ] **Step 3: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/references/hook-template.md
git commit -m "Add hook template with CLI and IDE hook format and conversion examples

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 6: skill-template.md — Kiro Skill Format (Near-1:1 Mapping)

This template documents the near-identical skill format and what needs changing.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/references/skill-template.md`
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (sections "Skills: Almost Identical", "Workflow 1: Single Skill Conversion")

**Content requirements:**
- Overview: Kiro adopted the same SKILL.md standard as Claude Code
- File locations: `.kiro/skills/<name>/` (workspace) and `~/.kiro/skills/<name>/` (global)
- Directory structure: `SKILL.md` (required), `scripts/` (optional), `references/` (optional)
- YAML frontmatter: `name` (required), `description` (required) — same fields as Claude
- Progressive loading: only name/description loaded at startup, full content on demand — same as Claude
- What changes during conversion:
  1. Tool names (Read → read, Bash → shell, etc.)
  2. File location (wherever stored → `.kiro/skills/<name>/`)
  3. Any Claude-specific tool references in instructions
- What stays the same: frontmatter format, markdown body, references/ and scripts/ subdirectories, auto-triggering via description matching
- Conversion example: Claude skill → Kiro skill (showing tool name translation and mapping comment header)
- Best practices: keep descriptions specific for auto-triggering, use scripts/ for executable logic, keep references/ for output format templates

- [ ] **Step 1: Write skill-template.md**

Create `claude-to-kiro/skills/claude-to-kiro/references/skill-template.md`.

- [ ] **Step 2: Verify skill-template.md completeness**

Check: file locations, directory structure, frontmatter fields, what changes vs stays the same, conversion example with tool name translation, best practices.

- [ ] **Step 3: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/references/skill-template.md
git commit -m "Add skill template documenting near-1:1 SKILL.md mapping

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 7: mcp-config-template.md — MCP Config Translation

The MCP config translation is the most mechanical conversion — same protocol, different file location and surrounding structure.

**Files:**
- Create: `claude-to-kiro/skills/claude-to-kiro/references/mcp-config-template.md`
- Reference: `claude-to-amazonq/skills/claude-to-amazonq/references/mcp-config-template.md` (structural template)
- Reference: `claude-to-kiro/docs/claude-to-kiro-design.md` (tool name translation, MCP config sections)

**Content requirements:**
- Overview: both tools use MCP, same protocol, different file locations
- Claude Code MCP config format (`~/.claude/settings.json` and `.claude/settings.json`)
- Kiro MCP config format (`.kiro/settings/mcp.json` workspace, `~/.kiro/settings/mcp.json` global)
- Translation steps:
  1. Copy `mcpServers` object from Claude settings
  2. Write to Kiro's `mcp.json` location
  3. Adjust paths referencing `~/.claude/` to Kiro equivalents or absolute paths
  4. Verify environment variables available in Kiro runtime
  5. For remote servers: confirm URL accessibility
- Config location mapping table (global, project, legacy)
- Conversion example: Claude settings.json → Kiro mcp.json (showing the direct copy + path adjustments)
- Permissions handling: Claude's `permissions` block has no direct Kiro equivalent at MCP level — document workaround via agent `allowedTools` and `toolsSettings`
- Known differences table (config file name, permissions location, hot reload, legacy format)
- Edge cases: Claude-specific MCP servers, authentication, path differences

- [ ] **Step 1: Read the amazonq MCP config template for structural reference**

Read `claude-to-amazonq/skills/claude-to-amazonq/references/mcp-config-template.md`.

- [ ] **Step 2: Write mcp-config-template.md**

Create `claude-to-kiro/skills/claude-to-kiro/references/mcp-config-template.md`. Follow the amazonq template structure but with Kiro file locations and any Kiro-specific differences.

- [ ] **Step 3: Verify mcp-config-template.md completeness**

Check: both Claude and Kiro formats shown, translation steps, config location mapping, conversion example, permissions handling, known differences, edge cases.

- [ ] **Step 4: Commit**

```bash
git add claude-to-kiro/skills/claude-to-kiro/references/mcp-config-template.md
git commit -m "Add MCP config template for Claude-to-Kiro translation

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 8: Update INDEX.md

Add the new skill to the repository's skill index.

**Files:**
- Modify: `INDEX.md`
- Reference: existing INDEX.md entries for format

- [ ] **Step 1: Read INDEX.md**

Read `INDEX.md` to understand the current format and entries.

- [ ] **Step 2: Add claude-to-kiro entry**

Add an entry following the existing format:
- Skill name: **claude-to-kiro**
- Description: Converts Claude Code configurations into AWS Kiro IDE/CLI equivalents using signal-based content classification and tool name translation. Supports four conversion workflows: single skill, full project, global config, and plugin-to-power bundle.
- Usage examples showing trigger phrases

- [ ] **Step 3: Commit**

```bash
git add INDEX.md
git commit -m "Add claude-to-kiro to skill index

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Task 9: Generate HTML Versions of Documentation

Generate styled HTML versions of the design document and plan for ease of reading and sharing. Use a dark theme consistent with the user's Cisco-themed documentation style.

**Files:**
- Create: `claude-to-kiro/docs/claude-to-kiro-design.html`
- Create: `claude-to-kiro/docs/claude-to-kiro-plan.html`
- Source: `claude-to-kiro/docs/claude-to-kiro-design.md`
- Source: `claude-to-kiro/docs/claude-to-kiro-plan.md`

**Content requirements:**
- Dark-themed design with clean typography, professional appearance
- Responsive layout that works on desktop and mobile
- Syntax-highlighted code blocks (markdown, JSON, bash, YAML)
- Proper rendering of all markdown tables, lists, and headings
- Navigation sidebar or table of contents for quick jumping between sections
- Attribution footer: "Created by: DT74 — Daren Threadingham"
- Self-contained HTML (inline CSS, no external dependencies)

- [ ] **Step 1: Generate design doc HTML**

Convert `claude-to-kiro/docs/claude-to-kiro-design.md` to a styled HTML document at `claude-to-kiro/docs/claude-to-kiro-design.html`. Include all content from the markdown, properly rendered with dark theme styling.

- [ ] **Step 2: Generate plan HTML**

Convert `claude-to-kiro/docs/claude-to-kiro-plan.md` to a styled HTML document at `claude-to-kiro/docs/claude-to-kiro-plan.html`. Include all content from the markdown, properly rendered with matching theme.

- [ ] **Step 3: Verify HTML renders correctly**

Open both HTML files in a browser to verify tables render correctly, code blocks are syntax-highlighted, navigation works, and the dark theme is consistent.

- [ ] **Step 4: Commit**

```bash
git add claude-to-kiro/docs/claude-to-kiro-design.html claude-to-kiro/docs/claude-to-kiro-plan.html
git commit -m "Add styled HTML versions of design doc and plan for sharing

Co-Authored-By: DT74 - Daren Threadingham <darent74@gmail.com>
Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
```

---

## Execution Order

Tasks 1-7 are independent of each other and can be executed in parallel.
Task 8 (INDEX.md update) can run in parallel with any other task.
Task 9 (HTML generation) depends on Tasks 1-8 being complete (needs final markdown content).

Recommended execution: parallel dispatch, or batch as:
- **Wave 1:** Tasks 1-4 (SKILL.md, mapping guide, steering template, agent template)
- **Wave 2:** Tasks 5-8 (hook template, skill template, MCP config template, INDEX.md)
- **Wave 3:** Task 9 (HTML generation — after all markdown is finalised)
