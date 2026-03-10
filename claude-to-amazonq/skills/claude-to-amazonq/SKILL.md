---
name: claude-to-amazonq
description: >
  This skill should be used when the user asks to "port Claude to Amazon Q",
  "convert Claude skills to Amazon Q", "migrate Claude config to Amazon Q",
  "export to Amazon Q Developer", "translate CLAUDE.md to Amazon Q rules",
  "convert MCP config for Amazon Q", or mentions migrating between Claude Code
  and Amazon Q Developer configurations.
version: 0.1.0
---

# Claude Code to Amazon Q Developer Migration

Convert Claude Code configurations (CLAUDE.md, skills, MCP servers, plugins) into
equivalent Amazon Q Developer configurations. Automatically classify content and
route it to the correct Amazon Q component.

## Important: Structural Differences

Claude Code and Amazon Q Developer organise customisation differently. A single Claude
skill maps to multiple Amazon Q components. This skill handles that translation.

| Claude Code | Amazon Q Developer | Notes |
|---|---|---|
| `CLAUDE.md` | `.amazonq/rules/*.md` | Project-level auto-loaded context |
| `~/.claude/CLAUDE.md` | `~/.aws/amazonq/prompts/*.md` | Global reusable prompts |
| Skills (`SKILL.md`) | Rules + Prompt Library + Custom Agents | Split by content type |
| MCP servers (`settings.json`) | `.amazonq/default.json` | Different JSON schema |
| Plugins | Custom Agents (JSON) | Partial mapping |
| Hooks | No equivalent | Document in migration report |
| Subagents | No equivalent | Document in migration report |

## Content Classification

Analyse each block of content from Claude source files and classify:

**Route to `.amazonq/rules/` (auto-loaded context):**
- Coding standards, conventions, architectural rules
- Phrases like "Always", "Never", "Prefer", "Use X over Y"
- Project structure descriptions, dependency notes
- Team conventions and constraints

**Route to `~/.aws/amazonq/prompts/` (on-demand reusable prompts):**
- Step-by-step workflows and procedures
- Numbered steps, decision trees, "First... then..." patterns
- Output format templates and search patterns
- Reusable prompt templates with placeholders

**Route to `.amazonq/default.json` (MCP configuration):**
- MCP server definitions, endpoints, authentication
- Tool integrations and API connections

**Route to custom agent JSON configs:**
- Agent definitions with permissions and tool access
- Plugin configurations with orchestration logic

**Route to `MIGRATION_REPORT.md` (not portable):**
- Hook logic (PreToolUse, PostToolUse, Stop)
- Subagent orchestration patterns
- Claude-specific features (context window management, skill triggering)

## Conversion Workflow

### Single Skill Conversion

1. Read the source skill's `SKILL.md` and all files in `references/`, `scripts/`, `examples/`
2. Classify each content block using the rules above
3. Generate Amazon Q output files:
   - Declarative content → `.amazonq/rules/<skill-name>-rules.md`
   - Procedural content → `prompts/<skill-name>.md`
   - Tool configs → merge into `.amazonq/default.json`
4. Inject `<!-- TODO: Manual migration needed — [description] -->` for non-portable content
5. Generate `MIGRATION_REPORT.md` entry

### Full Project Conversion

1. Read `CLAUDE.md` → generate `.amazonq/rules/project-context.md`
2. Scan `skills/` directory → convert each skill individually
3. Read MCP config from Claude settings → translate to `.amazonq/default.json`
4. Consolidate all migration report entries into single `MIGRATION_REPORT.md`

### Global Config Conversion

1. Read `~/.claude/CLAUDE.md` → generate global prompts
2. Scan `~/.claude/skills/*` → convert each to prompts + rules
3. Read global MCP config → translate to `~/.aws/amazonq/` format

## Output Modes

**Staging (default):** Write all output to `amazonq-export/` directory for review.

**Direct (when explicitly requested):** Write `.amazonq/` into current project root
and prompts to `~/.aws/amazonq/prompts/`. Only use when the user explicitly asks
to write in-place rather than to the staging directory.

## Amazon Q Rules Format

Consult `references/rules-template.md` for the `.amazonq/rules/` file format,
including priority levels, structure, and examples.

## Amazon Q Prompt Library Format

Consult `references/prompt-template.md` for the `~/.aws/amazonq/prompts/` file format
and naming conventions.

## Amazon Q Custom Agent Format

Consult `references/custom-agent-template.md` for the JSON schema and configuration
options for custom agents.

## MCP Config Translation

Consult `references/mcp-config-template.md` for translating Claude Code MCP server
configs (`settings.json`) to Amazon Q's `.amazonq/default.json` format.

## Migration Report

Always generate `MIGRATION_REPORT.md` in the output root containing:

1. **Successfully Ported** — checkbox list of converted items with source → target mapping
2. **Partial — Manual Adjustment Needed** — items with TODO comments injected
3. **Not Portable — Workarounds Suggested** — Claude features with no Amazon Q equivalent,
   including suggested alternative approaches
4. **File Mapping Table** — complete source → target mapping with status

For detailed classification heuristics and edge cases, consult `references/mapping-guide.md`.
