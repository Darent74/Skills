# Skills

A collection of reusable skills for Claude Code — each skill guides Claude's behaviour
through structured markdown and reference templates. No executable code — skills are
documentation-driven, teaching AI assistants how to perform complex tasks through
well-structured instructions.

## Skills

### Claude-to-Kiro

Converts Claude Code configurations into equivalent [AWS Kiro](https://kiro.dev) IDE/CLI
configurations. Kiro adopted many of Claude Code's primitives (SKILL.md format, hook event
model, MCP protocol), making most conversions near-direct.

**Four conversion workflows:**
- **Single skill** — one Claude skill → `.kiro/skills/` (near-1:1, tool names translated)
- **Full project** — CLAUDE.md + skills + hooks + MCP → complete `.kiro/` structure
- **Global config** — `~/.claude/` → `~/.kiro/` equivalents
- **Plugin-to-power** — plugin bundle → coordinated Kiro agents + steering + hooks + MCP

**Key features:**
- Signal-based content classification routes CLAUDE.md content to steering files with
  appropriate [inclusion modes](claude-to-kiro/docs/claude-to-kiro-design.md#for-users-new-to-kiro-how-the-mental-model-differs)
  (`always`, `fileMatch`, `manual`, `auto`)
- Automatic tool name translation (Claude `Read` → Kiro `read`, `Bash` → `shell`, etc.)
  with mapping comment headers for traceability
- Five-section migration report including Enhancement Opportunities — Kiro features you
  could adopt that Claude Code doesn't have
- [Plain-English migration guide](claude-to-kiro/docs/claude-to-kiro-design.md#for-users-new-to-kiro-how-the-mental-model-differs)
  for users new to Kiro covering rules, memory, security, skills, hooks, and agents

**Quick start:** `"Port my Claude skills to Kiro"` or `"Convert this project's Claude setup to Kiro"`

**Documentation:**
- [Design document](claude-to-kiro/docs/claude-to-kiro-design.md) — full architecture, concept mapping, worked examples
- [Design document (HTML)](claude-to-kiro/docs/claude-to-kiro-design.html) — styled version for easy reading
- [Implementation plan](claude-to-kiro/docs/claude-to-kiro-plan.md)

---

### Claude-to-AmazonQ

Converts Claude Code configurations into [Amazon Q Developer](https://aws.amazon.com/q/developer/)
equivalents. Uses signal-based content classification to split Claude's unified config
model across Amazon Q's component model (rules, prompts, custom agents, MCP config).

**Quick start:** `"Port my deployment skill to Amazon Q"` or `"Convert this project's Claude setup to Amazon Q"`

**Documentation:** [Design document](claude-to-amazonq/docs/claude-to-amazonq-design.md)

---

### Udemy Scanner

Searches and extracts structured course information from Udemy using multi-strategy
fallbacks (Class Central mirrors, WebSearch aggregation). Supports single course lookups,
multi-course comparisons, and topic-based bulk discovery.

**Quick start:** `"Search Udemy for Python networking courses"` or `"Compare these two Udemy courses"`

---

## Repo Structure

Each skill follows a standard layout:

```
<skill-name>/
├── docs/
│   └── <skill-name>-design.md    # Design document (problem, architecture, examples)
└── skills/
    └── <skill-name>/
        ├── SKILL.md               # Skill definition (metadata, workflows, triggers)
        └── references/            # Templates and mapping guides for output formats
```

See [INDEX.md](INDEX.md) for the complete skill catalog with function descriptions and
example trigger phrases.

## For New Users

These skills are designed for [Claude Code](https://claude.ai/code) (Anthropic's CLI).
Install a skill by copying its `skills/<skill-name>/` directory into your Claude Code
skills location (`~/.claude/skills/` for global, or a project's skills directory).

Each skill's `SKILL.md` contains trigger phrases in its YAML frontmatter — Claude
automatically activates the skill when your request matches those phrases. No manual
invocation needed.

## Contributing

When adding a new skill:

1. Create a directory at the repo root following the standard structure above
2. Write a design document in `docs/` before implementing the skill
3. Add a row to [INDEX.md](INDEX.md) with the skill name, directory, function summary,
   and example trigger phrases

---

*Created by: DT74 — Daren Threadingham*
