# Kiro Steering File Format

## Overview

Steering files are Kiro's equivalent of Claude Code's `CLAUDE.md`. Where Claude Code
puts all project rules, conventions, and workflows into a single file, Kiro splits
them across **multiple markdown files** in `.kiro/steering/`, each focused on one
concern.

The key difference is **inclusion modes**. Claude Code loads the entire `CLAUDE.md`
every interaction. Kiro lets you control *when* each steering file gets loaded —
always, only for certain file types, on-demand, or automatically when the request
matches a description. This reduces context noise and keeps the agent focused on
what matters for each task.

Steering files are plain markdown with a YAML frontmatter block that controls
loading behaviour. The body contains the same kind of instructions you would put
in `CLAUDE.md` — coding standards, architecture rules, security policies, workflows.

---

## File Locations

### Workspace Steering (Project-Scoped)

```
project-root/
└── .kiro/
    └── steering/
        ├── coding-standards.md
        ├── security-policies.md
        ├── react-conventions.md
        └── deployment-workflow.md
```

Workspace steering files apply only to the current project. This is where most
converted `CLAUDE.md` content lands.

### Global Steering (User-Scoped)

```
~/.kiro/
└── steering/
    ├── personal-preferences.md
    ├── global-security.md
    └── lessons.md
```

Global steering files apply to every project you open, equivalent to
`~/.claude/CLAUDE.md` or `~/.claude/tasks/lessons.md` in Claude Code.

### Precedence

When workspace and global steering files cover the same topic, **workspace takes
precedence**. This mirrors how project-level `CLAUDE.md` overrides global
`~/.claude/CLAUDE.md` in Claude Code.

---

## File Format

Every steering file is markdown with a YAML frontmatter block enclosed in `---`
fences. The frontmatter controls how and when the file is loaded.

### Minimal Format

```markdown
---
inclusion: always
---

# Topic Name

Your instructions here.
```

### Full Format (All Fields)

```markdown
---
inclusion: auto
name: deployment-checklist
description: Pre-deployment verification steps for production releases
---

# Deployment Checklist

Step-by-step instructions...
```

---

## Inclusion Modes

The `inclusion` field in the frontmatter determines when Kiro loads the steering
file into context. Choosing the right mode is the most important decision when
converting from `CLAUDE.md`.

### `inclusion: always` (Default)

Loaded every interaction, unconditionally. This is the closest behaviour to
`CLAUDE.md` — the content is always present in context.

```markdown
---
inclusion: always
---

# Project Architecture

- This is a Next.js 14 app with App Router
- Database is PostgreSQL via Prisma ORM
- All API routes must validate input with Zod schemas
- Never expose internal IDs in API responses — use UUIDs
```

**Use for:** Core project rules, architecture constraints, coding standards,
security policies — anything that applies regardless of what files are being
edited or what task is being performed.

### `inclusion: fileMatch`

Loaded only when the current interaction involves files matching a glob pattern.
Requires the `fileMatchPattern` field.

```markdown
---
inclusion: fileMatch
fileMatchPattern: "**/*.tsx"
---

# React Component Standards

- Use functional components exclusively
- Extract hooks into custom hook files when reused across 2+ components
- Co-locate component tests in __tests__/ adjacent to the component
- Use CSS Modules for styling — no inline styles except for dynamic values
```

**Use for:** Language-specific rules, framework conventions, file-type-specific
linting guidance. This mode keeps context clean by only loading rules when they
are relevant to the files being worked on.

**Pattern examples:**

| Pattern | Matches |
|---|---|
| `"**/*.tsx"` | All TSX files in any directory |
| `"**/*.{ts,tsx}"` | All TypeScript files |
| `"src/components/**"` | Everything under src/components/ |
| `"**/*.test.{ts,tsx}"` | All TypeScript test files |
| `"infrastructure/**/*.tf"` | Terraform files in infrastructure/ |

### `inclusion: manual`

Loaded only when explicitly requested by the user via `/` menu or `#steering-name`
reference in chat. Never loaded automatically.

```markdown
---
inclusion: manual
---

# Release Workflow

## Pre-Release Checks
1. Run full test suite: `npm run test:ci`
2. Check for uncommitted changes
3. Verify CHANGELOG.md is updated

## Version Bump
4. Run `npm version <major|minor|patch>`
5. Update lock file: `npm install`

## Publish
6. Push tags: `git push --follow-tags`
7. Create GitHub release from tag
8. Verify CI/CD pipeline completes
```

**Use for:** Step-by-step workflows, procedures you run occasionally, output
format templates, decision trees. These are things you do not want consuming
context every interaction — you pull them in when needed.

**Invocation:** Type `/` in chat to browse available manual steering, or
reference directly with `#release-workflow` (derived from the filename).

### `inclusion: auto`

Loaded when Kiro determines the steering file is relevant to the current request,
based on the `description` field. Requires both `name` and `description` fields.

```markdown
---
inclusion: auto
name: database-migrations
description: Guidelines for creating and running database migrations with Prisma
---

# Database Migration Guidelines

- Always create a migration for schema changes: `npx prisma migrate dev --name <descriptive-name>`
- Never edit migration files after they have been applied
- Include both up and down migration logic
- Test migrations against a copy of production data before deploying
- Add seed data updates in prisma/seed.ts when new required fields are added
```

**Use for:** Topic-specific guidance that should appear when relevant but not
every time. This is similar to how Claude Code skills auto-trigger based on
their description. Use `auto` when the content is too small or narrowly scoped
to justify a full skill directory, but too specialised for `always`.

---

## Foundational Templates

When you initialise Kiro in a project (`kiro init` or through the IDE), Kiro can
generate three foundational steering files that describe your project:

| File | Purpose |
|---|---|
| `product.md` | Product context — what the project does, who it serves, key features |
| `tech.md` | Technology stack — frameworks, languages, infrastructure, key dependencies |
| `structure.md` | Project structure — directory layout, module boundaries, key files |

These are generated with `inclusion: always` and provide the baseline context that
Kiro uses to understand your project. They are the equivalent of the "Project
Overview" and "Repository Structure" sections commonly found in `CLAUDE.md` files.

You can edit these files freely — they are standard steering files. The migration
skill does not overwrite them if they already exist.

---

## File References

Steering files can reference other files in the project using the file reference
syntax:

```markdown
#[[file:path/to/file]]
```

This tells Kiro to include the referenced file's content as additional context
when the steering file is loaded. Useful for pointing at config files, schema
definitions, or examples without duplicating their content.

**Examples:**

```markdown
#[[file:tsconfig.json]]
#[[file:prisma/schema.prisma]]
#[[file:.github/workflows/ci.yml]]
```

---

## Conversion Examples

### Example 1: Core Rules to `inclusion: always`

**Source (CLAUDE.md):**
```markdown
## Project Overview
This is a Next.js 14 app using App Router with PostgreSQL and Prisma.

## Code Style
- Always use TypeScript strict mode
- Prefer functional components over class components
- Never commit console.log statements
- Use Zod for all input validation
```

**Target (.kiro/steering/project-standards.md):**
```markdown
---
inclusion: always
---

# Project Standards

## Project Overview
This is a Next.js 14 app using App Router with PostgreSQL and Prisma.

#[[file:tsconfig.json]]

## Code Style
- Always use TypeScript strict mode
- Prefer functional components over class components
- Never commit console.log statements
- Use Zod for all input validation
```

**Why `always`:** These are universal rules — they apply to every interaction
regardless of file type or task. This matches the original `CLAUDE.md` behaviour
where the content was always present.

---

### Example 2: File-Type Rules to `inclusion: fileMatch`

**Source (CLAUDE.md):**
```markdown
## React Components
- Use functional components exclusively
- Extract reusable hooks into src/hooks/
- Co-locate tests with components in __tests__/
- Use CSS Modules — no styled-components or inline styles

## Python Scripts
- Follow PEP 8 strictly
- Type hints on all public functions
- Use pathlib instead of os.path
```

**Target (.kiro/steering/react-conventions.md):**
```markdown
---
inclusion: fileMatch
fileMatchPattern: "**/*.{tsx,jsx}"
---

# React Conventions

- Use functional components exclusively
- Extract reusable hooks into src/hooks/
- Co-locate tests with components in __tests__/
- Use CSS Modules — no styled-components or inline styles
```

**Target (.kiro/steering/python-conventions.md):**
```markdown
---
inclusion: fileMatch
fileMatchPattern: "**/*.py"
---

# Python Conventions

- Follow PEP 8 strictly
- Type hints on all public functions
- Use pathlib instead of os.path
```

**Why `fileMatch`:** These rules only matter when working with specific file
types. Loading React conventions while editing Python files wastes context.
The `fileMatchPattern` ensures rules appear only when relevant.

---

### Example 3: Workflow to `inclusion: manual`

**Source (CLAUDE.md):**
```markdown
## Database Migration Procedure
When I ask you to create a migration:
1. Review the current schema in prisma/schema.prisma
2. Make the requested schema changes
3. Run `npx prisma migrate dev --name <descriptive-name>`
4. Update prisma/seed.ts if new required fields were added
5. Run `npx prisma db seed` to verify seed data
6. Run `npx prisma generate` to update the client
7. Update any affected API routes or services
8. Run the test suite to verify nothing broke
```

**Target (.kiro/steering/database-migration-workflow.md):**
```markdown
---
inclusion: manual
---

# Database Migration Workflow

When creating a migration, follow these steps:

1. Review the current schema: #[[file:prisma/schema.prisma]]
2. Make the requested schema changes
3. Run `npx prisma migrate dev --name <descriptive-name>`
4. Update prisma/seed.ts if new required fields were added
5. Run `npx prisma db seed` to verify seed data
6. Run `npx prisma generate` to update the client
7. Update any affected API routes or services
8. Run the test suite to verify nothing broke
```

**Why `manual`:** This is a step-by-step procedure you invoke occasionally, not
something needed every conversation. Using `manual` keeps it out of context until
you request it with `/` or `#database-migration-workflow`.

---

### Example 4: Small Skill to `inclusion: auto`

When a Claude Code skill is too small to justify a full `.kiro/skills/<name>/`
directory (no `references/` or `scripts/` needed), convert it to a steering file
with `inclusion: auto` instead.

**Source (Claude Skill SKILL.md):**
```markdown
---
name: changelog-generator
description: Generate a CHANGELOG entry from recent git commits
---

## Workflow
1. Run `git log --oneline` from the last tag to HEAD
2. Group commits by type (feat, fix, chore, docs)
3. Format as a CHANGELOG.md entry under the new version heading
4. Prepend to CHANGELOG.md, preserving existing entries
```

**Target (.kiro/steering/changelog-generator.md):**
```markdown
---
inclusion: auto
name: changelog-generator
description: Generate a CHANGELOG entry from recent git commits
---

# Changelog Generator

1. Run `git log --oneline` from the last tag to HEAD
2. Group commits by type (feat, fix, chore, docs)
3. Format as a CHANGELOG.md entry under the new version heading
4. Prepend to CHANGELOG.md, preserving existing entries
```

**Why `auto`:** The original content was a skill that auto-triggered based on its
description. The `auto` inclusion mode preserves that behaviour — Kiro loads this
steering file when the user's request matches the description. Since there are no
reference files or scripts to bundle, a steering file is simpler than a full skill
directory.

---

## Best Practices

- **One concern per file** — Each steering file should cover a single topic or
  domain. Do not combine unrelated rules in one file. Smaller, focused files are
  easier to maintain and allow finer-grained inclusion mode control.

- **Keep files under 2,000 words** — Long steering files dilute the agent's
  attention. If a file grows beyond 2,000 words, split it into multiple files
  with appropriate inclusion modes.

- **Use `fileMatch` to reduce context** — Anything that only applies to specific
  file types should use `fileMatch`. This is one of Kiro's main advantages over
  `CLAUDE.md` — use it aggressively.

- **Set the right inclusion mode** — Default to `always` only for rules that
  genuinely apply to every interaction. Overloading `always` recreates the
  `CLAUDE.md` problem of loading everything every time.

- **Use file references instead of duplication** — Reference config files,
  schemas, and examples with `#[[file:path]]` rather than copying their content
  into the steering file. This keeps steering files short and ensures the agent
  sees current file contents.

- **Name files descriptively** — The filename (minus `.md`) becomes the reference
  name for manual steering (`#steering-name`). Use kebab-case names that clearly
  describe the topic: `react-conventions.md`, `security-policies.md`,
  `release-workflow.md`.

- **Provide `name` and `description` for `auto` mode** — Both fields are required
  for `auto` inclusion. Write the description as you would a skill trigger —
  specific enough to match relevant requests, broad enough not to miss them.
