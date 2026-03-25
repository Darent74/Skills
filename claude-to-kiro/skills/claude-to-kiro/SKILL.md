---
name: claude-to-kiro
description: >
  This skill should be used when the user asks to "port to Kiro",
  "convert skills to Kiro", "migrate config to Kiro", "export to Kiro",
  "translate CLAUDE.md to steering", "convert hooks to Kiro",
  "convert plugin to Kiro", or mentions migrating between Claude Code
  and Kiro IDE/CLI configurations.
version: 0.1.0
---

# Claude Code to Kiro Migration

Convert Claude Code configurations (CLAUDE.md, skills, hooks, MCP servers, plugins) into
equivalent Kiro IDE and CLI configurations. Automatically classify content and route it
to the correct Kiro component.

## Important: Structural Differences

Claude Code and Kiro organise customisation differently. This skill handles the translation
between the two models.

| Claude Code | Kiro | Notes |
|---|---|---|
| `CLAUDE.md` | `.kiro/steering/*.md` | Single file splits into multiple steering files with inclusion modes |
| Skills (`SKILL.md`) | `.kiro/skills/<name>/SKILL.md` | Near-1:1 — same standard, tool names translated |
| Hooks (PreToolUse, PostToolUse, Stop) | Agent config `hooks` block | Same event model, different config format (camelCase) |
| MCP servers (`settings.json`) | `.kiro/settings/mcp.json` | Same protocol, different file location and schema |
| Plugins | Agents + Steering + Skills + MCP | Decomposed into coordinated Kiro components |
| Subagents | `.kiro/agents/<name>.md` custom agents | Kiro supports persistent, named, parallel agents |
| Slash commands | `.kiro/steering/<name>.md` (`inclusion: manual`) | Invoked with `/` in chat or `#steering-name` reference |

## Content Classification

Analyse each block of content from Claude source files and classify into one of six
routing destinations:

**Route to `.kiro/steering/` (steering files):**
- Coding standards, conventions, architectural rules
- Phrases like "Always", "Never", "Prefer", "Use X over Y"
- Project structure descriptions, dependency notes
- Team conventions and constraints
- Security policies and guardrails
- File-type-specific rules (routed with `fileMatch` inclusion)
- Procedural workflows and step-by-step processes (routed with `manual` inclusion)

**Route to `.kiro/skills/<name>/` (agent skills):**
- Complete skill definitions with YAML frontmatter (`name`, `description`)
- Skill reference files, scripts, and examples
- Content that functions as a self-contained, triggerable capability

**Route to `.kiro/agents/<name>.md` (custom agents):**
- Subagent definitions with system prompts and tool access
- Plugin agent configurations with orchestration logic
- Agent-scoped hooks, MCP servers, and resource references

**Route to agent config `hooks` block (hooks):**
- PreToolUse, PostToolUse, Stop hook definitions
- Tool matchers, command restrictions, validation logic
- Event names translated to camelCase (preToolUse, postToolUse, stop)

**Route to `AGENTS.md` (project root):**
- CLAUDE.md directives that define agent behaviour, personas, or constraints
  compatible with the cross-tool AGENTS.md standard
- Always included by Kiro — no inclusion mode needed
- Generate only when agent-directive content is detected in CLAUDE.md

**Route to `MIGRATION_REPORT.md` (migration report):**
- Claude-specific features with no direct Kiro equivalent
- Ambiguous content that could map to multiple inclusion modes
- Enhancement opportunities noting Kiro-native features
- Items requiring manual review or adjustment

## Slash Command Routing

Claude Code slash commands map to Kiro manual steering files:

- Each slash command becomes `.kiro/steering/<command-name>.md`
- Set `inclusion: manual` so the user invokes it explicitly with `/` or `#steering-name`
- Preserve the command's procedural content in the steering file body
- Note the mapping in the migration report

## Steering Inclusion Mode Signals

When generating steering files from CLAUDE.md content, classify each file's inclusion
mode using these signal-based heuristics:

**`fileMatch`** — detected when content contains:
- Explicit file patterns: `*.tsx`, `*.py`, `.go files`
- Path references: "in src/components", "under tests/"
- Language scoping: "For TypeScript", "When writing Python", "React components"
- Requires `globs` field in YAML frontmatter (e.g., `globs: "**/*.{ts,tsx}"`)

**`manual`** — detected when content contains:
- Numbered step workflows, decision trees
- "First... then..." procedural patterns
- Slash command definitions
- Output format templates
- On-demand procedures the user invokes explicitly

**`auto`** — detected when content contains:
- Description-based matching phrases (similar to Claude skill triggering)
- Context-dependent rules that should load only when the request matches a description
- Content that was a standalone Claude skill but is too small for a full skill directory
- Requires `name` and `description` fields in the steering YAML frontmatter

**`always`** (default) — everything else:
- Architecture rules, coding standards, conventions
- Project structure, dependency notes
- Team constraints, security policies
- When no stronger signal is detected, default to `always`

**Ambiguous content handling:**
- When content could reasonably map to multiple inclusion modes, flag it in the
  migration report under "Partial -- Manual Adjustment Needed"
- Include the reasoning: which signals were detected and why the choice was unclear
- Inject `<!-- TODO: Review inclusion mode — could be [mode1] or [mode2]. [reasoning] -->`
  in the generated steering file

## Conversion Workflows

### Workflow 1: Single Skill Conversion

Convert one Claude Code skill to a Kiro skill.

1. Read the source skill's `SKILL.md` and all files in `references/`, `scripts/`, `examples/`
2. Translate tool names in all content (Read -> read, Bash -> shell, etc.)
3. Inject tool name mapping comment header in every affected file
4. Adjust YAML frontmatter -- keep `name` and `description`, drop Claude-specific fields
5. Copy `references/` and `scripts/` directories with tool names translated
6. Generate migration report entry

Consult `references/skill-template.md` for the `.kiro/skills/` output format.

### Workflow 2: Full Project Conversion

Convert an entire Claude Code project setup to Kiro.

1. Read `CLAUDE.md` -- split into steering files by topic, classify inclusion modes
2. Check for AGENTS.md-compatible directives in CLAUDE.md -- generate `AGENTS.md` if detected
3. Scan `skills/` directory -- convert each skill via Workflow 1
4. Read hook configurations -- translate to Kiro CLI hook format in agent configs
5. Read MCP config from Claude settings -- translate to `.kiro/settings/mcp.json`
6. Read any plugin definitions -- convert via Workflow 4
7. Consolidate migration report with enhancement opportunities

Consult `references/steering-template.md` for steering file format and inclusion modes.
Consult `references/hook-template.md` for CLI and IDE hook config format.
Consult `references/mcp-config-template.md` for MCP config translation.

### Workflow 3: Global Config Conversion

Convert the user's global Claude Code configuration to Kiro.

1. Read `~/.claude/CLAUDE.md` -- generate global steering files in `~/.kiro/steering/`
2. Scan `~/.claude/skills/*` -- convert each skill to `~/.kiro/skills/`
3. Read global MCP config from `~/.claude/settings.json` -- translate to `~/.kiro/settings/mcp.json`
4. Generate migration report

### Workflow 4: Plugin-to-Power Bundle

Convert a Claude Code plugin into a coordinated Kiro bundle of agents, steering,
hooks, and MCP. This is the most complex workflow.

1. Read `plugin.json` manifest -- extract name, description, component list
2. For each skill in the plugin -- convert via Workflow 1
3. For each agent definition -- generate `.kiro/agents/<name>.md` with:
   - `prompt` from agent system prompt
   - `tools` mapped from Claude tool permissions
   - `allowedTools` from auto-approved tools
   - `hooks` from Claude hook definitions (translated event names and matchers)
   - `mcpServers` from plugin MCP config
   - `resources` pointing to converted steering files and skills
4. For hook-only definitions (no parent agent) -- attach to a generated agent or document in steering
5. Merge MCP servers into `.kiro/settings/mcp.json`
6. Generate migration report with enhancement opportunities noting Kiro-only features

Consult `references/agent-template.md` for the `.kiro/agents/` markdown config format.

## Output Modes

**Staging (default):** Write all output to `kiro-export/` directory for review.

**Direct (when explicitly requested):** Write to `.kiro/` and `~/.kiro/` in-place.
Only use when the user explicitly asks to write in-place rather than to the staging
directory.

## Reference Templates

Consult the following reference files for detailed format specifications:

- Consult `references/mapping-guide.md` for complete concept mapping and classification heuristics
- Consult `references/steering-template.md` for `.kiro/steering/` file format and inclusion modes
- Consult `references/agent-template.md` for `.kiro/agents/` markdown config format
- Consult `references/hook-template.md` for CLI and IDE hook config format
- Consult `references/skill-template.md` for `.kiro/skills/` format (near-1:1 mapping)
- Consult `references/mcp-config-template.md` for `.kiro/settings/mcp.json` translation

## Migration Report

Always generate `MIGRATION_REPORT.md` in the output root containing five sections:

1. **Successfully Ported** -- checkbox list of converted items with source -> target mapping
   and inclusion mode where applicable
2. **Partial -- Manual Adjustment Needed** -- items with TODO comments injected, including
   ambiguous inclusion mode classifications with reasoning
3. **Not Portable -- Workarounds Suggested** -- Claude features with no Kiro equivalent,
   including suggested alternative approaches
4. **File Mapping Table** -- complete source -> target mapping with inclusion mode and status
5. **Enhancement Opportunities** -- Kiro-native features the user could adopt that have no
   Claude Code equivalent (fileMatch steering, agent knowledge bases, toolsSettings path
   restrictions, Kiro specs, IDE hooks, autonomous agent)

Inject `<!-- TODO: Manual migration needed -- [description] -->` comments in generated
files where manual adjustment is required.

For detailed classification heuristics and edge cases, consult `references/mapping-guide.md`.
