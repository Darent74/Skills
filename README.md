# Skills

A collection of reusable skills for Claude Code — each skill guides Claude's behavior through structured markdown and reference templates.

## What's in this repo

- **Claude-to-AmazonQ** — Converts Claude Code configurations (skills, hooks, MCP servers) into Amazon Q Developer equivalents
- **Udemy Scanner** — Searches and extracts structured Udemy course information using multi-strategy fallbacks
- **INDEX.md** — Full skill catalog with function descriptions and usage examples

## Repo structure

Each skill follows a standard layout:

```
<skill-name>/
├── docs/              # Design documents
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md       # Skill definition and workflows
│       └── references/    # Templates and mapping guides
└── scripts/           # Optional automation
```

See [INDEX.md](INDEX.md) for the complete skill directory.
