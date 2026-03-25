# Kiro Custom Agent Configuration

## Overview

Kiro custom agents are persistent, named agent definitions that live in `.kiro/agents/`
as individual markdown files. Each agent has its own system prompt, tool permissions,
MCP server access, resources, hooks, and optional IDE shortcuts.

Custom agents are the Kiro equivalent of two Claude Code concepts:

- **Claude plugins** — Bundled configurations that group skills, agents, and MCP
  servers together. In Kiro, a plugin decomposes into one or more custom agents
  plus associated steering files, skills, and MCP config.
- **Claude subagents** — Ephemeral task-specific agents spawned inline via the
  `Agent` tool. In Kiro, subagents become persistent custom agent definitions that
  the user (or another agent) can activate by name.

The key difference is persistence: Claude subagents are ephemeral and defined inline
at spawn time. Kiro custom agents are pre-defined configurations stored on disk that
persist across sessions. This makes them discoverable, shareable, and version-controlled.

---

## File Locations

### Workspace Agents (Project-Scoped)

```
project-root/
└── .kiro/
    └── agents/
        ├── code-reviewer.md
        ├── deployment.md
        └── network-automation.md
```

Workspace agents apply only to the current project. This is where most converted
Claude subagent definitions and plugin agents land.

### Global Agents (User-Scoped)

```
~/.kiro/
└── agents/
    ├── general-assistant.md
    ├── security-auditor.md
    └── documentation-writer.md
```

Global agents are available in every project you open. Use these for role-based
agents that are not project-specific — a general code reviewer, a documentation
writer, a security auditor.

### Precedence

When workspace and global agents share the same filename, **workspace takes
precedence**. This mirrors how project-level configuration overrides global
configuration throughout Kiro.

---

## Configuration Reference

Agent files are markdown with a YAML frontmatter block enclosed in `---` fences.
The frontmatter defines the agent's configuration. The markdown body (after the
closing `---`) is ignored by the parser — use it for human-readable notes about
the agent if desired.

### Minimal Configuration

```markdown
---
name: code-reviewer
description: Reviews code changes for quality, consistency, and potential bugs
prompt: >
  You are a code reviewer. Focus on correctness, readability, and adherence
  to the project's coding standards. Flag potential bugs, security issues,
  and performance concerns. Be specific in your feedback.
---
```

### Full Configuration (All 14 Fields)

```markdown
---
name: network-automation
description: Specialised agent for Cisco network automation and configuration management
prompt: file://prompts/network-automation-system-prompt.md
model: claude-sonnet-4
tools:
  - read
  - write
  - shell
  - "@netbox-server"
allowedTools:
  - fs_read
  - fs_search
  - shell_execute:npm test*
mcpServers:
  netbox-server:
    command: npx
    args: ["-y", "@netbox/mcp-server"]
    env:
      NETBOX_URL: "https://netbox.internal.example.com"
      NETBOX_TOKEN: "${NETBOX_API_TOKEN}"
includeMcpJson: true
resources:
  - file://docs/network-topology.md
  - file://.kiro/steering/network-standards.md
  - skill://network-scanner
  - knowledgeBase:
      path: docs/runbooks/
toolsSettings:
  shell_execute:
    allowedCommands:
      - "npm test*"
      - "npm run build*"
      - "python scripts/*.py"
      - "ansible-playbook*"
    deniedCommands:
      - "rm -rf*"
      - "chmod*"
      - "sudo*"
  fs_write:
    allowedPaths:
      - "src/"
      - "configs/"
      - "docs/"
  fs_read:
    allowedPaths:
      - "*"
hooks:
  agentSpawn:
    - command: echo "Agent network-automation activated"
  preToolUse:
    - matcher: shell_execute
      command: echo "Shell command requested"
  postToolUse:
    - matcher: fs_write
      command: npm run lint --fix
  stop:
    - command: echo "Agent session complete"
toolAliases:
  run: shell_execute
  search: fs_search
keyboardShortcut: ctrl+shift+n
welcomeMessage: >
  Network automation agent ready. I can help with device configurations,
  topology queries, and automated network tasks. What would you like to do?
---
```

---

## Field Reference

### 1. `name`

**Type:** `string` (required)
**Purpose:** Unique identifier for the agent. Used in activation commands, keyboard
shortcuts, and inter-agent references.

```yaml
name: code-reviewer
```

**Naming conventions:**
- Use `kebab-case`
- Keep names short and descriptive
- Must be unique within the scope (workspace or global)

---

### 2. `description`

**Type:** `string` (required)
**Purpose:** Human-readable summary of what the agent does and when to use it.
Displayed in the agent picker and used by other agents to understand capabilities.

```yaml
description: Reviews code changes for quality, consistency, and potential bugs
```

---

### 3. `prompt`

**Type:** `string` (required)
**Purpose:** The system prompt that defines the agent's behaviour, personality, and
constraints. Accepts either an inline string or a `file://` URI pointing to an
external markdown file.

**Inline string:**
```yaml
prompt: >
  You are a senior code reviewer. Focus on correctness, readability,
  and adherence to project standards. Be specific in your feedback.
```

**External file reference:**
```yaml
prompt: file://prompts/code-reviewer-system-prompt.md
```

Use `file://` for longer prompts to keep agent configs readable. The referenced
file path is relative to the project root.

---

### 4. `model`

**Type:** `string` (optional)
**Purpose:** The AI model this agent uses. When omitted, the agent uses Kiro's
default model.

```yaml
model: claude-sonnet-4
```

**Common values:** `claude-sonnet-4`, `claude-haiku-4`. Enterprise deployments
may restrict available models via model governance policies.

---

### 5. `tools`

**Type:** `string[]` (optional)
**Purpose:** Array of tools the agent can access. Controls which capabilities
the agent has at a coarse level.

```yaml
tools:
  - read
  - write
  - shell
  - aws
  - "@netbox-server"
  - "*"
```

**Built-in tool identifiers:**

| Value | Grants Access To |
|---|---|
| `"read"` | `fs_read`, `fs_list`, `fs_search` — file reading and searching |
| `"write"` | `fs_write`, `fs_edit` — file creation and modification |
| `"shell"` | `shell_execute` — shell command execution |
| `"aws"` | AWS service tools (Kiro-specific) |
| `"@server_name"` | All tools from a named MCP server |
| `"*"` | All available tools (use with caution) |

When `tools` is omitted, the agent inherits the default tool set.

---

### 6. `allowedTools`

**Type:** `string[]` (optional)
**Purpose:** Tools that run without a permission prompt. By default, Kiro asks
for user confirmation before certain tool actions. Tools listed here are
auto-approved. Supports glob patterns.

```yaml
allowedTools:
  - fs_read
  - fs_search
  - shell_execute:npm test*
  - shell_execute:npm run lint*
```

**Glob pattern examples:**

| Pattern | Auto-Approves |
|---|---|
| `fs_read` | All file reads |
| `shell_execute:npm *` | Any npm command |
| `shell_execute:git status` | Exactly `git status` |
| `mcp__netbox__*` | All tools from the netbox MCP server |

This is the Kiro equivalent of Claude Code's `settings.json` allow list, but
scoped to a specific agent rather than applied globally.

---

### 7. `toolsSettings`

**Type:** `object` (optional)
**Purpose:** Per-tool configuration that restricts what specific tools can do.
More granular than `tools` or `allowedTools` — controls *which files* a tool
can access and *which commands* the shell can run.

```yaml
toolsSettings:
  shell_execute:
    allowedCommands:
      - "npm test*"
      - "npm run build*"
      - "python scripts/*.py"
    deniedCommands:
      - "rm -rf*"
      - "chmod 777*"
      - "sudo*"
  fs_write:
    allowedPaths:
      - "src/"
      - "tests/"
  fs_read:
    allowedPaths:
      - "*"
```

**Sub-fields:**

| Field | Applies To | Purpose |
|---|---|---|
| `allowedPaths` | `fs_read`, `fs_write`, `fs_edit` | Restrict file access to specific directories |
| `allowedCommands` | `shell_execute` | Whitelist of permitted shell commands (glob patterns) |
| `deniedCommands` | `shell_execute` | Blacklist of blocked shell commands (glob patterns) |

**Evaluation order:** `deniedCommands` takes precedence over `allowedCommands`.
If a command matches both, it is denied.

This has no direct Claude Code equivalent. Claude Code uses global permission
settings and hook-based validation. Kiro's `toolsSettings` provides the same
protection with less setup, scoped per agent.

---

### 8. `mcpServers`

**Type:** `object` (optional)
**Purpose:** MCP servers available to this agent. Defines server connections
inline within the agent config.

```yaml
mcpServers:
  netbox-server:
    command: npx
    args: ["-y", "@netbox/mcp-server"]
    env:
      NETBOX_URL: "https://netbox.internal.example.com"
      NETBOX_TOKEN: "${NETBOX_API_TOKEN}"
  github-server:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_TOKEN: "${GITHUB_TOKEN}"
```

Each key is the server name (referenced in `tools` as `"@server_name"`). The
value follows the standard MCP server configuration format: `command`, `args`,
and `env`.

Environment variables use `${VAR_NAME}` syntax for secret injection.

---

### 9. `includeMcpJson`

**Type:** `boolean` (optional, default `false`)
**Purpose:** When `true`, the agent inherits MCP servers defined in the workspace
(`.kiro/settings/mcp.json`) and global MCP configurations, in addition to any
servers defined inline in `mcpServers`.

```yaml
includeMcpJson: true
```

When `false` (the default), the agent only has access to MCP servers explicitly
defined in its own `mcpServers` block. This provides isolation — agents that
should not access certain external services can be locked down by omitting
`includeMcpJson`.

---

### 10. `resources`

**Type:** `array` (optional)
**Purpose:** Files, skills, and knowledge bases the agent can reference for
context. Resources are loaded into the agent's context when activated.

```yaml
resources:
  - file://docs/architecture.md
  - file://.kiro/steering/security-policies.md
  - skill://network-scanner
  - knowledgeBase:
      path: docs/runbooks/
```

**Resource types:**

| URI Scheme | Purpose | Example |
|---|---|---|
| `file://` | Include a specific file's content | `file://docs/api-spec.yaml` |
| `skill://` | Include a skill's full content | `skill://network-scanner` |
| `knowledgeBase` | Index a directory for RAG-style retrieval | `knowledgeBase: { path: "docs/" }` |

**`file://` paths** are relative to the project root. Use these to give the agent
access to documentation, schemas, config files, or steering files without loading
them globally.

**`skill://` references** load the named skill's `SKILL.md` and associated resources
into the agent's context. This is how agents gain specialised capabilities.

**`knowledgeBase` objects** index a directory so the agent can search and retrieve
relevant documents. This is a Kiro-native feature with no Claude Code equivalent —
Claude Code loads referenced files in full, whereas knowledge bases enable
selective retrieval from large document sets.

---

### 11. `hooks`

**Type:** `object` (optional)
**Purpose:** Lifecycle event hooks scoped to this agent. Hooks run shell commands
or agent prompts at specific points in the agent's execution.

```yaml
hooks:
  agentSpawn:
    - command: echo "Agent activated at $(date)"
  preToolUse:
    - matcher: shell_execute
      command: echo "About to run shell command"
    - matcher: fs_write
      prompt: "Verify this write follows project conventions"
  postToolUse:
    - matcher: fs_write
      command: npm run lint --fix
  stop:
    - command: npm test
```

**Trigger events:**

| Event | When It Fires |
|---|---|
| `agentSpawn` | When the agent is activated |
| `preToolUse` | Before a tool executes (can block) |
| `postToolUse` | After a tool executes |
| `stop` | When the agent finishes its task |

**Hook entry fields:**

| Field | Purpose |
|---|---|
| `matcher` | Tool name to match (required for `preToolUse` and `postToolUse`) |
| `command` | Shell command to run |
| `prompt` | Agent prompt to evaluate (alternative to `command`) |

Claude Code hooks (`PreToolUse`, `PostToolUse`, `Stop`) map directly to Kiro
hook events (`preToolUse`, `postToolUse`, `stop`). Kiro adds `agentSpawn` as
an additional event with no Claude Code equivalent.

---

### 12. `toolAliases`

**Type:** `object` (optional)
**Purpose:** Remap tool names within this agent's scope. Useful for creating
shorter or more intuitive tool references in prompts and hooks.

```yaml
toolAliases:
  run: shell_execute
  search: fs_search
  read: fs_read
  edit: fs_edit
```

Aliases only affect this agent — they do not change tool names globally. This
is the Kiro equivalent of Amazon Q's `toolAliases` field, which serves the same
purpose.

---

### 13. `keyboardShortcut`

**Type:** `string` (optional, IDE only)
**Purpose:** Keyboard shortcut to quickly switch to this agent in the Kiro IDE.
Has no effect in the Kiro CLI.

```yaml
keyboardShortcut: ctrl+shift+r
```

**Common patterns:**
- `ctrl+shift+<letter>` — standard modifier combination
- Keep shortcuts unique across all agents to avoid conflicts

This has no Claude Code equivalent — Claude Code does not have agent switching
via keyboard shortcuts.

---

### 14. `welcomeMessage`

**Type:** `string` (optional)
**Purpose:** Message displayed when the agent is activated. Introduces the
agent's capabilities and provides guidance on what the user can ask.

```yaml
welcomeMessage: >
  Code reviewer ready. Share a diff, file, or PR link and I'll review it
  for quality, correctness, and adherence to project standards.
```

This has no Claude Code equivalent — Claude Code subagents do not display
welcome messages.

---

## Conversion Examples

### Example 1: Claude Plugin to Kiro Agent

A Claude Code plugin bundles skills, agents, and MCP servers in a `plugin.json`
manifest. In Kiro, the plugin decomposes into individual custom agents plus
associated steering, skills, and MCP config.

**Source (Claude Plugin `plugin.json`):**
```json
{
  "name": "network-tools",
  "description": "Network automation plugin for Cisco environments",
  "skills": ["network-scanner", "config-validator"],
  "agents": ["bulk-config-agent"],
  "mcp": {
    "netbox": {
      "command": "npx",
      "args": ["-y", "@netbox/mcp-server"]
    }
  }
}
```

**Target (`.kiro/agents/network-tools.md`):**
```markdown
---
name: network-tools
description: Network automation agent for Cisco environments — includes scanning and config validation
prompt: >
  You are a network automation specialist for Cisco environments. You have
  access to network scanning and configuration validation capabilities.
  Always validate device configs before applying changes. Use NETCONF/RESTCONF
  over SSH where supported. Follow enterprise security policies.
tools:
  - read
  - write
  - shell
  - "@netbox"
allowedTools:
  - fs_read
  - fs_search
mcpServers:
  netbox:
    command: npx
    args: ["-y", "@netbox/mcp-server"]
resources:
  - skill://network-scanner
  - skill://config-validator
  - file://.kiro/steering/network-standards.md
hooks:
  preToolUse:
    - matcher: shell_execute
      prompt: "Verify this command is safe to run on network devices"
welcomeMessage: >
  Network tools agent ready. I can scan network devices, validate
  configurations, and manage bulk config operations via NetBox.
---
```

**What happened:**
- Plugin `name` and `description` → agent `name` and `description`
- Plugin `skills` → agent `resources` with `skill://` references
- Plugin `agents` content → merged into the agent's `prompt`
- Plugin `mcp` → agent `mcpServers`
- Hook permissions from the plugin → agent `hooks` and `allowedTools`

---

### Example 2: Claude Subagent to Kiro Custom Agent

Claude Code subagents are spawned inline with the `Agent` tool, passing a
system prompt and task description. In Kiro, these become persistent custom
agent definitions.

**Source (Claude subagent spawn in CLAUDE.md or skill):**
```markdown
## Research Subagent

When performing research tasks, spawn a subagent with:
- System prompt: "You are a research assistant. Search the web and codebase
  to find relevant information. Summarise findings concisely with source links."
- Tools: Read, Grep, WebSearch, WebFetch
- Task: The specific research question
```

**Target (`.kiro/agents/researcher.md`):**
```markdown
<!-- Tool names translated from Claude Code -> Kiro:
     Read -> read (fs_read), Grep -> search (fs_search)
     WebSearch -> web_search, WebFetch -> web_fetch -->

---
name: researcher
description: Research assistant that searches the web and codebase to answer questions
prompt: >
  You are a research assistant. Search the web and codebase to find relevant
  information. Summarise findings concisely with source links. Prioritise
  authoritative sources and cross-reference claims across multiple sources.
tools:
  - read
  - shell
model: claude-sonnet-4
allowedTools:
  - fs_read
  - fs_search
  - web_search
  - web_fetch
welcomeMessage: >
  Research agent ready. Ask me a question and I'll search the codebase
  and web to find the answer.
---
```

**What happened:**
- Subagent system prompt → agent `prompt`
- Subagent tool list → agent `tools` and `allowedTools` (with tool name translation)
- Inline spawn pattern → persistent named agent file
- Added `welcomeMessage` (Kiro enhancement — not present in Claude Code)
- Added `model` specification (Kiro allows per-agent model selection)

---

## What Cannot Be Converted

### Ephemeral Subagent Spawning Patterns

Claude Code's `Agent` tool supports spawning subagents dynamically at runtime with
arbitrary system prompts and task descriptions. The agent is created on the fly,
executes its task, returns results, and is discarded.

Kiro custom agents are **pre-defined** — they must exist as files on disk before
they can be activated. There is no equivalent to:

```
Use the Agent tool to spawn a subagent with prompt "..." and task "..."
```

**Workaround:** Identify recurring subagent spawn patterns and convert each into
a named custom agent. One-off or truly dynamic spawns cannot be converted — document
these in the migration report as manual setup items.

### Inline Agent Definitions

Claude Code allows defining agent behaviour inline within skills, CLAUDE.md, or
even conversation context. For example, a skill might say "spawn an agent that
does X with tools Y" as part of a workflow step.

Kiro requires agent definitions to be separate files in `.kiro/agents/`. Inline
definitions must be extracted into their own agent files.

**Workaround:** Extract each inline agent definition into a `.kiro/agents/<name>.md`
file and replace the inline reference with a note to activate that agent.

### Dynamic Tool Permissions

Claude Code subagents can be spawned with different tool permissions each time
based on the task. A skill might spawn a "read-only" subagent for research and
a "read-write" subagent for implementation.

Kiro agents have fixed tool permissions defined in their configuration file. To
achieve variable permissions, create multiple agents with different tool sets
(e.g., `researcher-readonly.md` and `researcher-readwrite.md`).

---

## Best Practices

- **One domain per agent** — Each agent should have a clear, focused responsibility.
  A "code-reviewer" agent should review code, not also deploy or write documentation.
  Focused agents are easier to reason about, test, and maintain.

- **Use resources for context** — Instead of embedding long instructions in the
  `prompt` field, use `resources` to point to steering files, skills, and knowledge
  bases. This keeps agent configs readable and lets you share context across agents.

- **Restrict tools appropriately** — Start with the minimum tool set an agent needs
  and expand only when required. Use `toolsSettings` to add path and command
  restrictions on top of the base tool permissions. A code reviewer should not need
  `shell` access; a deployment agent should not need `write` access to source code.

- **Name clearly** — Agent names appear in the activation picker and keyboard
  shortcuts. Use descriptive `kebab-case` names: `code-reviewer`, `network-automation`,
  `security-auditor`. Avoid generic names like `helper` or `agent-1`.

- **Use `file://` for long prompts** — If a system prompt exceeds 10-15 lines,
  move it to an external file with `prompt: file://prompts/<name>.md`. This keeps
  agent configs scannable and makes prompts easier to edit.

- **Scope MCP servers per agent** — Use `includeMcpJson: false` (the default) to
  isolate agents from workspace MCP servers they do not need. Only set
  `includeMcpJson: true` for agents that genuinely need access to all configured
  servers.

- **Add welcome messages** — A good welcome message helps users (and other agents)
  understand what the agent can do. Include the agent's capabilities and suggest
  example requests.

- **Version-control agent configs** — Since agents are plain markdown files in
  `.kiro/agents/`, they are naturally version-controlled. Commit them alongside
  your code so team members get the same agent configurations.

---

*Last updated: March 2026*
