# MCP Config Translation — Claude Code to Amazon Q Developer

## Overview

Both Claude Code and Amazon Q Developer support MCP (Model Context Protocol) servers,
but use different JSON schemas to configure them. This guide covers translating between
the two formats.

## Claude Code MCP Config

Claude Code stores MCP server configs in `~/.claude/settings.json` or project-level
`.claude/settings.json`:

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

## Amazon Q Developer MCP Config

Amazon Q stores MCP config in `.amazonq/default.json` (project-level) or
`~/.aws/amazonq/default.json` (global):

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

## Translation Process

The MCP server schema is largely compatible between Claude Code and Amazon Q Developer.
The primary differences are **file location** and **surrounding config structure**.

### Step-by-Step

1. Read the `mcpServers` object from Claude's `settings.json`
2. Copy it directly into the Amazon Q `default.json` under the same `mcpServers` key
3. Adjust any file paths that reference Claude-specific locations:
   - `~/.claude/` paths → update to Amazon Q equivalent or absolute paths
   - Project-relative paths generally work as-is
4. Verify environment variables are available in the Amazon Q environment
5. For remote (HTTP/SSE) servers: confirm the URL is accessible from the Amazon Q runtime

### Config Location Mapping

| Scope | Claude Code | Amazon Q Developer |
|---|---|---|
| Global | `~/.claude/settings.json` | `~/.aws/amazonq/default.json` |
| Project | `.claude/settings.json` | `.amazonq/default.json` |
| Legacy | — | `.amazonq/mcp.json` (deprecated) |

### Conversion Example

**Source (`~/.claude/settings.json`):**
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
    "allow": ["Read", "Write"],
    "deny": ["Bash"]
  }
}
```

**Target (`.amazonq/default.json`):**
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

**Migration notes for this example:**
- `mcpServers` block copied directly — schema is compatible
- `permissions` block has no Amazon Q equivalent — documented in migration report
  - Workaround: use custom agent `tools.allowed` / `tools.denied` for similar restrictions

## Known Differences

| Feature | Claude Code | Amazon Q Developer |
|---|---|---|
| Server schema | Compatible | Compatible |
| Config file name | `settings.json` | `default.json` |
| Environment variable syntax | `${VAR}` | `${VAR}` (same) |
| Permissions alongside MCP | In same file | Separate (agent config) |
| Legacy format support | No | Yes (`mcp.json` with `useLegacyMcpJson`) |
| Hot reload | Restart required | Restart required |

## Edge Cases

- **Claude-specific MCP servers** (e.g., servers that depend on Claude's tool names
  like `Read`, `Write`, `Bash`) may not work in Amazon Q. Document these in the
  migration report with alternatives.
- **Authentication** — Verify that API keys and tokens stored in environment variables
  are also available in the Amazon Q IDE environment.
- **Path differences** — macOS/Linux paths are generally compatible. Windows paths
  may need adjustment if the target environment differs.
