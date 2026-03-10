# Amazon Q Developer — Custom Agent Configuration

## Overview

Amazon Q Developer custom agents are JSON-configured specialised agents that define
behaviour, permissions, available tools, and MCP server access. They are the closest
equivalent to Claude Code plugins.

## File Location

Custom agent configs are defined in `.amazonq/default.json` or project-level config:

```
project-root/
└── .amazonq/
    └── default.json
```

## JSON Schema

### Basic Agent Configuration

```json
{
  "customAgents": [
    {
      "name": "agent-name",
      "description": "What this agent does and when to use it",
      "instructions": "Detailed behavioural instructions for the agent",
      "tools": {
        "allowed": ["tool1", "tool2"],
        "denied": ["tool3"]
      },
      "mcpServers": ["server-name-1"],
      "resourceContext": {
        "files": ["path/to/file.md"],
        "directories": ["src/"]
      }
    }
  ]
}
```

### Full Agent Configuration

```json
{
  "customAgents": [
    {
      "name": "network-automation",
      "description": "Specialised agent for Cisco network automation tasks",
      "instructions": "Focus on network automation using PyATS, Netmiko, and NAPALM. Follow enterprise security policies. Always validate device configs before applying. Use NETCONF/RESTCONF over SSH where supported.",
      "tools": {
        "allowed": [
          "readFile",
          "writeFile",
          "executeCommand",
          "webSearch"
        ],
        "denied": [
          "deleteFile"
        ]
      },
      "mcpServers": [
        "dnac-server",
        "netbox-server"
      ],
      "resourceContext": {
        "files": [
          ".amazonq/rules/network-standards.md",
          "docs/network-topology.md"
        ],
        "directories": [
          "src/automation/",
          "configs/"
        ]
      },
      "toolAliases": {
        "run": "executeCommand",
        "read": "readFile"
      }
    }
  ]
}
```

## Converting from Claude Code Plugins

### Plugin → Custom Agent Mapping

| Claude Plugin Field | Amazon Q Agent Field |
|---|---|
| `plugin.json` → `name` | `name` |
| `plugin.json` → `description` | `description` |
| Skills content (merged) | `instructions` |
| MCP server configs | `mcpServers` |
| Hook permissions | `tools.allowed` / `tools.denied` |
| Agent definitions | Merge into `instructions` |

### Conversion Example

**Source (Claude Plugin `plugin.json`):**
```json
{
  "name": "network-tools",
  "description": "Network automation plugin for Cisco environments",
  "skills": ["network-scanner", "config-validator"],
  "agents": ["bulk-config-agent"]
}
```

**Target (Amazon Q custom agent in `.amazonq/default.json`):**
```json
{
  "customAgents": [
    {
      "name": "network-tools",
      "description": "Network automation agent for Cisco environments — includes scanning and config validation",
      "instructions": "...(merged from skill SKILL.md files)...",
      "mcpServers": ["...(from plugin MCP config)..."],
      "resourceContext": {
        "files": [
          ".amazonq/rules/network-scanner-rules.md",
          ".amazonq/rules/config-validator-rules.md"
        ]
      }
    }
  ]
}
```

### What Cannot Be Converted

- **Claude hooks** → No agent equivalent. Document as manual setup in migration report.
  Suggested workaround: use `tools.denied` for restriction-type hooks.
- **Subagent spawning** → Amazon Q agents cannot spawn sub-agents. Document as limitation.
  Suggested workaround: create multiple specialised agents.
- **Skill triggering logic** → Amazon Q agents are invoked explicitly, not auto-triggered.
  Document trigger phrases in agent description for discoverability.

## Best Practices

- **Keep instructions focused** — One domain per agent, not a catch-all
- **Use resource context** — Point agents to relevant rules and documentation files
- **Restrict tools appropriately** — Use `tools.denied` for safety-critical operations
- **Name clearly** — Agent names should indicate their specialisation
- **Reference rules files** — Link to `.amazonq/rules/` in `resourceContext.files`
  so the agent inherits project standards automatically
