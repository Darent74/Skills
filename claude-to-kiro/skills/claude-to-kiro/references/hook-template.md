# Kiro Hook Configuration

## Overview

Kiro hooks are lifecycle event handlers that execute custom logic at specific points
during an agent's operation. They let you intercept tool calls, validate output, enforce
safety guardrails, and trigger follow-up actions — the same use cases as Claude Code hooks,
with a direct event model mapping.

Kiro supports two distinct hook systems:

- **CLI agent hooks** — Defined in agent configuration files, scoped to a specific agent.
  These fire on agent lifecycle events (tool use, agent spawn, stop) and execute shell
  commands. This is the direct mapping target for Claude Code hooks.
- **IDE file-event hooks** — Configured through the Kiro IDE UI or via natural language.
  These fire on file system events (save, create, delete) and can trigger agent prompts
  or shell commands. These have no Claude Code equivalent — they are a Kiro-only
  enhancement opportunity.

---

## CLI Agent Hooks

### Event Types

Kiro CLI hooks support five event types. Each event fires at a specific point in the
agent lifecycle:

| Event | When It Fires | Common Use Cases |
|---|---|---|
| `agentSpawn` | When the agent starts | Environment validation, dependency checks, workspace setup |
| `userPromptSubmit` | When the user sends a message (before the agent processes it) | Input validation, prompt logging, context injection |
| `preToolUse` | Before a tool executes | Block dangerous commands, validate arguments, enforce policies |
| `postToolUse` | After a tool executes | Validate output, log results, trigger follow-up actions |
| `stop` | When the agent finishes its response | Cleanup, summary generation, notification |

### Configuration Format

CLI hooks are defined inside the `hooks` object of an agent configuration file
(`.kiro/agents/<name>.md`). Each event type holds an array of hook definitions:

```yaml
hooks:
  preToolUse:
    - matcher: "shell_execute"
      command: "/path/to/validate-command.sh"
      timeout_ms: 10000
    - matcher: "fs_write"
      command: "/path/to/validate-write.sh"
  postToolUse:
    - matcher: "*"
      command: "/path/to/log-tool-use.sh"
  agentSpawn:
    - command: "/path/to/setup-env.sh"
  userPromptSubmit:
    - command: "/path/to/validate-input.sh"
  stop:
    - command: "/path/to/cleanup.sh"
```

### Matcher Field

The `matcher` field determines which tool invocations trigger the hook. It supports
multiple matching strategies:

| Matcher Type | Example | Matches |
|---|---|---|
| Canonical tool name | `fs_read` | The `fs_read` tool only |
| Alias | `read` | The `read` tool (alias for `fs_read`) |
| MCP tool pattern | `@git/status` | The `status` tool from the `git` MCP server |
| Wildcard | `*` | Any tool invocation |

You can use either the canonical name or the alias — Kiro resolves both. MCP tool
patterns use the `@server/tool` syntax to match tools from specific MCP servers.

The `matcher` field is required for `preToolUse` and `postToolUse` events (which are
tool-scoped). It is not used for `agentSpawn`, `userPromptSubmit`, or `stop` events
(which are lifecycle-scoped).

### STDIN Input

When a hook command executes, it receives a JSON object via STDIN containing event
context:

```json
{
  "event": "preToolUse",
  "tool": {
    "name": "shell_execute",
    "input": {
      "command": "rm -rf /tmp/build"
    }
  },
  "cwd": "/home/user/project",
  "agent": "deployment"
}
```

The JSON structure varies by event type:

- **`preToolUse` / `postToolUse`** — includes `tool.name`, `tool.input`, and for
  `postToolUse` also `tool.output`
- **`agentSpawn`** — includes `agent` name and `cwd`
- **`userPromptSubmit`** — includes `prompt` (the user's message) and `cwd`
- **`stop`** — includes `agent` name and `cwd`

### Exit Codes

| Exit Code | Meaning | Applies To |
|---|---|---|
| `0` | Success — proceed normally | All events |
| `2` | Block — prevent the tool from executing | `preToolUse` only |
| Non-zero (other) | Hook failure — agent logs warning, proceeds | All events |

Exit code `2` is the safety mechanism. When a `preToolUse` hook exits with code 2,
the tool call is blocked and the agent receives the hook's STDERR output as an
explanation of why the tool was blocked.

### Timeout and Caching

| Field | Default | Description |
|---|---|---|
| `timeout_ms` | `30000` (30 seconds) | Maximum time the hook command can run before being killed |
| `cache_ttl_seconds` | — | Optional. Cache the hook result for this many seconds to avoid re-running on repeated identical calls |

```yaml
hooks:
  preToolUse:
    - matcher: "shell_execute"
      command: "/path/to/policy-check.sh"
      timeout_ms: 5000
      cache_ttl_seconds: 60
```

---

## IDE File-Event Hooks

IDE hooks are a Kiro-only feature with no Claude Code equivalent. They are configured
through the Kiro IDE UI or by describing what you want in natural language. They fire
on file system and interaction events rather than agent lifecycle events.

### Triggers

| Trigger | Description |
|---|---|
| File saved | A file matching a glob pattern is saved |
| File created | A new file matching a glob pattern is created |
| File deleted | A file matching a glob pattern is deleted |
| User prompt submit | The user sends a message in the chat |
| Agent stop | The agent finishes its response |
| Manual trigger | The user explicitly activates the hook |

### Actions

Each IDE hook performs one of two actions:

- **Ask Kiro** — sends a prompt to the agent. Use this for intelligent follow-up
  ("when I save a test file, suggest missing test cases").
- **Run Command** — executes a shell command. Use this for deterministic automation
  ("when I save a .py file, run `black` on it").

### File Pattern Matching

IDE hooks use glob patterns to scope which files trigger the hook:

```
**/*.test.ts       — any TypeScript test file
src/components/**  — any file under src/components/
*.py               — any Python file in the root
**/*.{css,scss}    — any CSS or SCSS file
```

### Configuration

IDE hooks can be configured in two ways:

1. **Via UI** — the Kiro IDE provides a hook configuration panel where you select
   the trigger, file pattern, and action
2. **Via natural language** — describe what you want in chat ("when I save a test
   file, run the linter on it") and Kiro creates the hook

IDE hooks are stored in the project's `.kiro/` directory and are version-controllable.

---

## Event Name Translation: Claude Code to Kiro

| Claude Code Event | Kiro CLI Event | Notes |
|---|---|---|
| `PreToolUse` | `preToolUse` | PascalCase to camelCase. Same semantics. |
| `PostToolUse` | `postToolUse` | PascalCase to camelCase. Same semantics. |
| `Stop` | `stop` | Same semantics. |
| — | `agentSpawn` | Kiro-only. No Claude Code equivalent. |
| — | `userPromptSubmit` | Kiro-only. No Claude Code equivalent. |
| `SessionStart` | — | Claude-only. See "What Cannot Be Converted" below. |
| `SessionEnd` | — | Claude-only. See "What Cannot Be Converted" below. |

---

## Tool Matcher Translation: Claude Code to Kiro

When converting Claude Code hook matchers, translate tool names to Kiro equivalents:

| Claude Code Matcher | Kiro Matcher | Notes |
|---|---|---|
| `Bash` | `shell_execute` or `shell` | Either canonical or alias works |
| `Read` | `fs_read` or `read` | |
| `Write` | `fs_write` or `write` | |
| `Edit` | `fs_edit` or `edit` | |
| `Glob` | `fs_list` or `list` | |
| `Grep` | `fs_search` or `search` | |
| `WebFetch` | `web_fetch` | No alias |
| `WebSearch` | `web_search` | No alias |
| `*` (wildcard) | `*` | Same syntax |
| MCP tool (`mcp__server__tool`) | `@server/tool` | Different syntax for MCP tools |

---

## Conversion Examples

### Example 1: Block Dangerous Shell Commands

**Claude Code** — `PreToolUse` hook in `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "python3 /home/user/scripts/block-dangerous-commands.py",
        "timeout_ms": 5000
      }
    ]
  }
}
```

Where `block-dangerous-commands.py` reads STDIN, checks if the command contains
`rm -rf`, and exits with code 2 if so:

```python
import sys, json
data = json.load(sys.stdin)
cmd = data.get("tool_input", {}).get("command", "")
if "rm -rf" in cmd:
    print("Blocked: rm -rf is not allowed", file=sys.stderr)
    sys.exit(2)
sys.exit(0)
```

**Kiro CLI** — `preToolUse` hook in `.kiro/agents/deployment.md`:

```yaml
hooks:
  preToolUse:
    - matcher: "shell_execute"
      command: "python3 /home/user/scripts/block-dangerous-commands.py"
      timeout_ms: 5000
```

The hook script itself needs one adjustment — the STDIN JSON field name changes from
`tool_input` to `tool.input`:

```python
import sys, json
data = json.load(sys.stdin)
cmd = data.get("tool", {}).get("input", {}).get("command", "")
if "rm -rf" in cmd:
    print("Blocked: rm -rf is not allowed", file=sys.stderr)
    sys.exit(2)
sys.exit(0)
```

**What changed:**
- Event name: `PreToolUse` to `preToolUse`
- Matcher: `Bash` to `shell_execute`
- Hook location: global `settings.json` to agent-scoped config
- STDIN JSON structure: `tool_input` to `tool.input` (nested)

### Example 2: Log Tool Output

**Claude Code** — `PostToolUse` hook:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "command": "bash /home/user/scripts/log-tool-output.sh"
      }
    ]
  }
}
```

**Kiro CLI** — `postToolUse` hook in `.kiro/agents/code-reviewer.md`:

```yaml
hooks:
  postToolUse:
    - matcher: "*"
      command: "bash /home/user/scripts/log-tool-output.sh"
      timeout_ms: 10000
```

**What changed:**
- Event name: `PostToolUse` to `postToolUse`
- Wildcard matcher `*` is unchanged
- Hook location: global to agent-scoped
- Explicit `timeout_ms` added (recommended — Claude uses implicit defaults)

---

## What Cannot Be Converted

### IDE File-Event Hooks (No Claude Source)

Kiro IDE hooks (file saved/created/deleted triggers) have no Claude Code equivalent.
There is nothing to convert from. These are flagged as **enhancement opportunities**
in the migration report:

```markdown
## 5. Enhancement Opportunities
- **IDE hooks**: Consider adding file-event hooks for auto-linting (on save),
  test generation (on create), or cleanup (on delete). These complement CLI
  hooks with no Claude Code equivalent.
```

### SessionStart / SessionEnd (Different Model)

Claude Code supports `SessionStart` and `SessionEnd` hook events. Kiro does not have
direct equivalents for these session-boundary events. The closest approximations:

| Claude Code | Kiro Approximation | Fidelity |
|---|---|---|
| `SessionStart` | `agentSpawn` hook | **Partial** — fires when an agent starts, not when a session opens. Multiple agents may spawn per session. |
| `SessionEnd` | `stop` hook | **Partial** — fires when an agent finishes a response, not when the session closes. May fire multiple times per session. |

These are flagged as **partial conversions** in the migration report with a note
explaining the semantic difference.

---

## Best Practices

1. **Scope hooks to agents** — Attach hooks to the specific agent that needs them
   rather than duplicating hooks across all agents. A deployment agent needs the
   `rm -rf` blocker; a code-review agent that only reads files does not.

2. **Use specific matchers over wildcards** — `shell_execute` is better than `*`
   when you only need to guard shell commands. Wildcards add overhead to every tool
   call and make debugging harder.

3. **Keep timeouts reasonable** — The default 30-second timeout is generous. Most
   validation hooks should complete in under 5 seconds. Set `timeout_ms` explicitly
   to catch runaway scripts early.

4. **Use `cache_ttl_seconds` for expensive checks** — If a hook calls an external
   API or runs a slow validation, cache the result to avoid repeated identical checks
   during a single agent session.

5. **Write to STDERR for block messages** — When a `preToolUse` hook exits with
   code 2, the agent receives STDERR as the explanation. Write a clear, actionable
   message ("Blocked: rm -rf is not allowed by project policy") rather than a
   cryptic error.

6. **Test hooks independently** — Hook scripts receive JSON via STDIN. Test them
   outside of Kiro by piping sample JSON: `echo '{"event":"preToolUse",...}' | ./hook.sh`

7. **Prefer `preToolUse` over `postToolUse` for safety** — If the goal is to prevent
   damage, block before execution. Post-execution hooks can log and alert but cannot
   undo the action.

---

*Reference template for the claude-to-kiro migration skill.*
