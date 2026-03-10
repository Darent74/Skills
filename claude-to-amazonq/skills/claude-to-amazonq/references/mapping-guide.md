# Claude Code to Amazon Q Developer — Complete Mapping Guide

## Concept Mapping

### Configuration Files

| Claude Code | Location | Amazon Q Developer | Location |
|---|---|---|---|
| `CLAUDE.md` (project) | Project root | Project rules | `.amazonq/rules/*.md` |
| `CLAUDE.md` (global) | `~/.claude/CLAUDE.md` | Prompt library | `~/.aws/amazonq/prompts/*.md` |
| `settings.json` (MCP) | `~/.claude/settings.json` | MCP config | `.amazonq/default.json` |
| Plugin config | `.claude-plugin/plugin.json` | Custom agent config | `.amazonq/agents/*.json` |

### Feature Mapping

| Claude Code Feature | Amazon Q Equivalent | Mapping Quality |
|---|---|---|
| CLAUDE.md instructions | `.amazonq/rules/*.md` | Direct — auto-loaded context |
| Skill SKILL.md | Rules + Prompts (split) | Requires classification |
| Skill `references/` | Prompt library or rules | Content-dependent |
| Skill `scripts/` | No equivalent | Document in report |
| Skill `examples/` | Include in prompts | Inline into prompt files |
| MCP servers | MCP servers | Direct — different JSON schema |
| Hooks (PreToolUse) | Custom agent guardrails | Partial — different mechanism |
| Hooks (PostToolUse) | No equivalent | Document in report |
| Hooks (Stop) | No equivalent | Document in report |
| Subagents | No equivalent | Document in report |
| Plugins | Custom agents | Partial — different architecture |
| Slash commands | Prompt library `@name` | Similar invocation pattern |
| Context window management | Workspace indexing | Different mechanism |

### Content Classification Rules

**Declarative content → Rules:**
- Contains: "Always", "Never", "Prefer", "Must", "Do not"
- Describes: architecture, conventions, file structure, dependencies
- Establishes: standards, constraints, team agreements
- Pattern: `<imperative verb> + <standard/convention>`

**Procedural content → Prompts:**
- Contains: numbered steps, "First", "Then", "Next", "Finally"
- Describes: workflows, processes, decision trees
- Includes: output format templates, search patterns
- Pattern: `Step N: <action>` or `To <achieve X>, <do Y>`

**Tool content → MCP/Agent config:**
- Contains: server URLs, endpoints, authentication
- Describes: tool capabilities, API connections
- Includes: environment variables, connection strings

**Non-portable content → Migration report:**
- References: Claude-specific APIs (subagent spawning, hook events)
- Depends on: Claude's context window, skill triggering system
- Uses: Claude-specific tool names (Read, Write, Edit, Bash, Glob, Grep)

### Priority Mapping

Claude Code has no explicit priority system. When converting to Amazon Q rules,
assign priority based on language strength:

| Claude Language | Amazon Q Priority |
|---|---|
| "MUST", "NEVER", "CRITICAL" | Critical |
| "Always", "Never", "Required" | High |
| "Prefer", "Should", "Recommended" | Medium |
| "Consider", "May", "Optional" | Low |

### Naming Conventions

**Claude Code:**
- Skills: `kebab-case` directory names (`my-skill/`)
- Files: `SKILL.md`, `CLAUDE.md` (uppercase)

**Amazon Q Developer:**
- Rules: `kebab-case.md` in `.amazonq/rules/`
- Prompts: `kebab-case.md` in prompts directory (no spaces)
- Agents: `kebab-case.json` in agent config

**Conversion:** Transform Claude skill names to Amazon Q naming:
- `my-skill/SKILL.md` → `my-skill-rules.md` (rules) + `my-skill.md` (prompts)
