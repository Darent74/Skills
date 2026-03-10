# Skill Index

A catalog of all skills in this repository. Update this table when adding or removing skills.

## Skills

| Skill Name | Directory | Function | Example Usage |
|---|---|---|---|
| Claude-to-AmazonQ | `claude-to-amazonq/` | Converts Claude Code configurations (skills, CLAUDE.md, hooks, MCP servers) into Amazon Q Developer equivalents (rules, prompts, custom agents, MCP config). Uses signal-based content classification to route each piece of content to the correct Amazon Q component. | "Port my deployment skill to Amazon Q", "Convert this project's Claude setup to Amazon Q", "Migrate Claude config to Amazon Q Developer" |
| Udemy Scanner | `udemy-scanner/` | Searches and extracts structured course information from Udemy using multi-strategy fallbacks (Class Central mirrors, WebSearch aggregation). Supports single course lookups, multi-course comparisons, and topic-based bulk discovery. | "Search Udemy for Python networking courses", "Compare these two Udemy courses", "Find the best Kubernetes courses on Udemy" |

## Adding a New Skill

When you add a new skill to this repository:

1. Create a directory at the repo root following the standard structure:
   ```
   <skill-name>/
   ├── docs/                    # Optional: design documents
   ├── skills/
   │   └── <skill-name>/
   │       ├── SKILL.md         # Skill definition
   │       └── references/      # Templates and guides
   └── scripts/                 # Optional: automation scripts
   ```
2. Add a row to the **Skills** table above with the skill name, directory, function summary, and example trigger phrases.
