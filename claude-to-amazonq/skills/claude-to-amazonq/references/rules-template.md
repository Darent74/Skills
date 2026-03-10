# Amazon Q Developer — Rules File Format

## Overview

Amazon Q Developer rules live in `.amazonq/rules/` within the project root.
They are automatically loaded and applied as context during chat interactions.

## File Location

```
project-root/
└── .amazonq/
    └── rules/
        ├── coding-standards.md
        ├── architecture-guidelines.md
        ├── security-policies.md
        └── team-conventions.md
```

## File Format

Rules files are Markdown with a structured format. Each file should focus on a
single domain or concern.

### Template

```markdown
# Rule Name

## Purpose
Brief explanation of why this rule exists and what it enforces.

## Priority
Critical | High | Medium | Low

## Instructions

### Category 1
- Specific directive 1
- Specific directive 2

### Category 2
- Specific directive 3
- Specific directive 4

## Error Handling
Fallback strategies when rules conflict or cannot be applied.

## Examples

### Correct
<code example demonstrating compliance>

### Incorrect
<code example demonstrating violation>
```

## Priority Levels

| Priority | When to Use | Behaviour |
|---|---|---|
| **Critical** | Security, data integrity, breaking changes | Always enforced, overrides other rules |
| **High** | Architecture patterns, coding standards | Strongly enforced, rarely overridden |
| **Medium** | Style preferences, best practices | Applied when no conflict exists |
| **Low** | Suggestions, optional improvements | Applied when explicitly relevant |

## Example: Converted from Claude CLAUDE.md

**Source (Claude Code CLAUDE.md):**
```markdown
## Code Style
- Always use TypeScript strict mode
- Prefer functional components over class components
- Never commit console.log statements
```

**Target (Amazon Q `.amazonq/rules/coding-standards.md`):**
```markdown
# Coding Standards

## Purpose
Enforce consistent code quality and style across the project.

## Priority
High

## Instructions

### TypeScript
- Enable and maintain TypeScript strict mode in all tsconfig files
- Resolve all strict mode violations before committing

### React Components
- Write functional components exclusively
- Do not use class components for new code
- Migrate class components to functional when modifying existing files

### Code Hygiene
- Remove all console.log statements before committing
- Use a proper logging framework for production logging

## Error Handling
When strict mode causes build failures in third-party type definitions,
add targeted type assertions rather than disabling strict mode.
```

## Example: Converted from Claude Skill (Declarative Content)

**Source (Claude Skill SKILL.md — declarative section):**
```markdown
## Important: Udemy Blocks Direct Fetching
Udemy returns 403 Forbidden on direct WebFetch requests. Never attempt to fetch
udemy.com URLs directly — it will always fail.
```

**Target (Amazon Q `.amazonq/rules/udemy-scanner-rules.md`):**
```markdown
# Udemy Access Rules

## Purpose
Prevent failed HTTP requests to Udemy and enforce alternative retrieval strategies.

## Priority
Critical

## Instructions

### URL Restrictions
- Never fetch udemy.com URLs directly via HTTP — returns 403 Forbidden
- Use web search strategies instead of direct page fetching
- Prefer Class Central mirrors for structured course metadata

## Error Handling
If a Udemy URL is encountered, search for the course via web search engines
or Class Central instead of attempting direct access.
```

## Best Practices

- **One concern per file** — Keep rules focused on a single domain
- **Be specific** — Vague rules produce inconsistent results
- **Include examples** — Show correct and incorrect patterns
- **Set appropriate priority** — Critical for things that break, Low for preferences
- **Keep files under 2,000 words** — Longer files dilute effectiveness
