# MCP Config Translation — Claude Code to Kiro

## Overview

Both Claude Code and Kiro use MCP (Model Context Protocol) for external tool
integrations. The underlying protocol is identical — same JSON-RPC transport,
same server schema, same `mcpServers` object structure. The only differences
are **file location** and **how permissions are handled** around MCP tools.

This makes MCP config translation the most mechanical part of the migration:
copy the server definitions, write them to the Kiro location, and adjust any
Claude-specific paths.

## Claude Code MCP Config

Claude Code stores MCP server configs in two scopes:

- **Global:** `~/.claude/settings.json`
- **Project:** `.claude/settings.json`

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"],
      "env": {
        "API_KEY": "value"
      }
    },
    "remote-server": {
      "url": "https://mcp.example.com/sse",
      "headers": {
        "Authorization": "Bearer token"
      }
    }
  },
  "permissions": {
    "allow": ["Read", "Write", "mcp__server-name__tool"],
    "deny": ["Bash"]
  }
}
```

The `permissions` block sits alongside `mcpServers` in the same file. MCP tool
permissions use the `mcp__<server>__<tool>` naming convention.

## Kiro MCP Config

Kiro stores MCP server configs in two scopes:

- **Workspace (project):** `.kiro/settings/mcp.json`
- **Global:** `~/.kiro/settings/mcp.json`

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"],
      "env": {
        "API_KEY": "value"
      }
    },
    "remote-server": {
      "url": "https://mcp.example.com/sse",
      "headers": {
        "Authorization": "Bearer token"
      }
    }
  }
}
```

The `mcpServers` object schema is identical to Claude Code. No field renaming
or restructuring is needed for the server definitions themselves.

## Translation Steps

### Step-by-Step

1. **Copy the `mcpServers` object** from Claude's `settings.json`
2. **Write to the Kiro location:**
   - Global: `~/.claude/settings.json` → `~/.kiro/settings/mcp.json`
   - Project: `.claude/settings.json` → `.kiro/settings/mcp.json`
3. **Adjust `~/.claude/` paths** — any server args or env values that reference
   `~/.claude/` directories should be updated to absolute paths or Kiro equivalents
4. **Verify environment variables** are available in the Kiro runtime environment
   (IDE or CLI) — env vars set in shell profiles work for both, but IDE-specific
   env var injection may differ
5. **Confirm remote server URLs** are accessible from the Kiro runtime — same
   network, same auth tokens, same SSE/HTTP endpoints

### Config Location Mapping

| Scope | Claude Code | Kiro |
|---|---|---|
| Global | `~/.claude/settings.json` | `~/.kiro/settings/mcp.json` |
| Project | `.claude/settings.json` | `.kiro/settings/mcp.json` |

Note: Kiro uses a dedicated `mcp.json` file rather than embedding MCP config
inside a broader `settings.json`. The Kiro file contains **only** the
`mcpServers` object (and optionally `mcpPolicy` for enterprise governance).

## Conversion Example

### Source: `~/.claude/settings.json`

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/daren/projects"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  },
  "permissions": {
    "allow": ["Read", "Write", "mcp__github__search_repositories"],
    "deny": ["Bash"]
  }
}
```

### Target: `~/.kiro/settings/mcp.json`

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/daren/projects"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### Migration Notes for This Example

- **`mcpServers` block copied directly** — schema is fully compatible
- **`permissions` block has no direct Kiro equivalent at MCP level** — see
  Permissions Handling below
- **No path adjustments needed** — no server args reference `~/.claude/`
- **`${GITHUB_TOKEN}` env var** — verify this is set in the Kiro environment

## Permissions Handling

Claude Code's `permissions` block (which sits alongside `mcpServers` in
`settings.json`) has **no direct Kiro equivalent at the MCP config level**.

Kiro handles tool restrictions through **per-agent configuration** instead:

### Workaround: Agent `allowedTools` and `toolsSettings`

In Kiro agent configs (`.kiro/agents/<name>.md`), you can restrict which tools
an agent may use and how:

**`allowedTools`** — whitelist of tools the agent can call:

```yaml
allowedTools:
  - read
  - write
  - edit
  - mcp__github__search_repositories
```

**`toolsSettings`** — granular per-tool restrictions:

```yaml
toolsSettings:
  shell:
    allowedCommands:
      - "npm test"
      - "npm run build"
    deniedCommands:
      - "rm -rf"
      - "chmod"
  read:
    allowedPaths:
      - "src/**"
      - "tests/**"
    deniedPaths:
      - ".env"
      - "secrets/**"
```

### Translation from Claude Permissions

| Claude Code `permissions` | Kiro Equivalent | Location |
|---|---|---|
| `allow: ["Read", "Write"]` | `allowedTools: [read, write]` | Agent config |
| `deny: ["Bash"]` | Omit `shell` from `allowedTools` | Agent config |
| `allow: ["mcp__server__tool"]` | `allowedTools: [mcp__server__tool]` | Agent config |
| Blanket allow/deny | Per-agent, per-tool granularity | Agent config |

The migration report flags every `permissions` entry and shows the suggested
Kiro agent config equivalent. If no custom agents exist, a default agent config
is generated to carry the permission restrictions.

## Known Differences

| Feature | Claude Code | Kiro |
|---|---|---|
| Config file name | `settings.json` (shared with permissions) | `mcp.json` (dedicated) |
| Server schema | Compatible | Compatible |
| Environment variable syntax | `${VAR}` | `${VAR}` (same) |
| Permissions location | Same file (`permissions` block) | Separate (agent config `allowedTools` / `toolsSettings`) |
| Hot reload | Restart required | Restart required |
| Legacy format support | No | No |
| Enterprise MCP governance | No | Yes (`mcpPolicy` for registry-approved servers) |
| MCP tool naming | `mcp__<server>__<tool>` | `mcp__<server>__<tool>` (same) |

## Edge Cases

### Claude-Specific MCP Servers

MCP servers that depend on Claude Code's tool names (e.g., servers that invoke
`Read`, `Write`, `Bash` by name in their prompts or logic) may not work in Kiro
where the canonical tool names differ (`read`/`fs_read`, `write`/`fs_write`,
`shell`/`shell_execute`). Document these in the migration report and flag for
manual review.

### Authentication

API keys and tokens stored in environment variables must also be available in
the Kiro environment. Check:

- Shell profile env vars (`.zshrc`, `.bashrc`) — generally work for both
- IDE-injected env vars — may differ between Claude Code (terminal) and Kiro IDE
- Token refresh mechanisms — OAuth tokens with Claude-specific refresh flows
  will need Kiro-compatible alternatives

### Path Differences

- **`~/.claude/` paths in server args** — update to absolute paths or remove
  the Claude-specific prefix. Common example: MCP servers that store data in
  `~/.claude/mcp-data/` should be pointed to a tool-agnostic location.
- **Project-relative paths** — generally work as-is since both tools operate
  from the project root
- **Cross-platform paths** — macOS/Linux paths are compatible between Claude
  Code and Kiro. Windows paths may need adjustment if switching environments.

### Servers with Inline Config

Some MCP servers accept configuration via args or env that references Claude
Code conventions. Examples:

- Servers configured with `--config ~/.claude/mcp-configs/server.json`
- Servers using env vars like `CLAUDE_PROJECT_ROOT` that Kiro does not set

These require manual adjustment — the migration report flags any arg or env
value containing `claude` (case-insensitive) for review.

---

*Template version: 0.1.0 | March 2026*
