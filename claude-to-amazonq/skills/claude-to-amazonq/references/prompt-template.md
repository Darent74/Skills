# Amazon Q Developer — Prompt Library Format

## Overview

The Amazon Q Developer prompt library stores reusable prompts that engineers invoke
with `@prompt-name` in the chat interface. Prompts live in `~/.aws/amazonq/prompts/`
(global, user-wide) and are available across all projects.

## File Location

```
~/.aws/amazonq/
└── prompts/
    ├── udemy-scanner.md
    ├── code-review-checklist.md
    └── network-audit.md
```

## Naming Conventions

- **No spaces** in filenames — use hyphens or underscores
- The filename (without `.md`) becomes the `@` reference name
- Example: `udemy-scanner.md` → invoked as `@udemy-scanner`

## File Format

Prompt files are plain Markdown. No frontmatter is required. The content is injected
as context when the user invokes the prompt.

### Template

```markdown
# Prompt Title

## Context
Brief description of what this prompt does and when to use it.

## Instructions

Step 1: [First action]
Step 2: [Second action]
Step 3: [Third action]

## Output Format

[Describe or template the expected output structure]

## Constraints

- [Constraint 1]
- [Constraint 2]
```

## Example: Converted from Claude Skill (Procedural Content)

**Source (Claude Skill SKILL.md — procedural section):**
```markdown
## Workflow

### Single Course Lookup

1. Identify the course URL or title
2. Extract the course slug
3. Run Strategy 1 (Class Central) and Strategy 2 (WebSearch) in parallel
4. If instructor is unknown, run Strategy 3
5. Merge results, resolve conflicts by preferring Class Central data
6. Output in the structured format above
```

**Target (`~/.aws/amazonq/prompts/udemy-scanner.md`):**
```markdown
# Udemy Course Scanner

## Context
Search for and extract structured information from Udemy courses. Udemy blocks
direct HTTP access (403), so use alternative retrieval strategies.

## Instructions

### Single Course Lookup

Step 1: Identify the course URL or title from the user's request
Step 2: Extract the course slug from the URL (e.g., "python-for-network-engineers")
Step 3: Search Class Central for the course: site:classcentral.com udemy "<slug-keywords>"
Step 4: Search web for additional details: udemy "<course title>" instructor hours rating
Step 5: If instructor is unknown, search for instructor subdomain patterns
Step 6: Merge results — prefer Class Central data when sources conflict

### Bulk Search

Step 1: Search for courses on the topic: udemy best courses "<topic>" <current-year>
Step 2: Search Class Central: site:classcentral.com udemy "<topic>"
Step 3: Collect top 5-10 results
Step 4: Run single-course lookups on top candidates
Step 5: Present as a ranked list with key differentiators

## Output Format

| Field | Detail |
|---|---|
| **Platform** | Udemy |
| **Instructor** | Name |
| **Duration** | X hours |
| **Rating** | X.X / 5 |
| **URL** | [Link](url) |

## Constraints

- Never fetch udemy.com directly — always use search strategies
- Mark unavailable fields as "Not available" — never fabricate data
- Prefer results from the current and previous year
```

## Example: Converted from Claude CLAUDE.md (Workflow Section)

**Source (Claude CLAUDE.md):**
```markdown
## Workflow
1. Plan First: Write plan to tasks/todo.md
2. Verify Plan: Check in before starting
3. Track Progress: Mark items complete
4. Document Results: Add review section
```

**Target (`~/.aws/amazonq/prompts/task-workflow.md`):**
```markdown
# Task Workflow

## Context
Standard workflow for managing development tasks in this project.

## Instructions

Step 1: Write a plan to tasks/todo.md with checkable items
Step 2: Verify the plan — check in before starting implementation
Step 3: Track progress — mark items complete as work proceeds
Step 4: Explain changes — provide a high-level summary at each step
Step 5: Document results — add a review section to tasks/todo.md

## Constraints

- Do not skip the planning step for non-trivial tasks
- Always verify the plan before implementation begins
```

## Usage in Amazon Q

Engineers invoke prompts in the chat interface:

```
@udemy-scanner Find me a PyATS course on Udemy
@task-workflow Set up the new monitoring feature
@code-review-checklist Review the changes in @workspace
```

Prompts can be combined with workspace context:
```
@network-audit using the files in @src/configs/
```

## Best Practices

- **Keep prompts focused** — One workflow or task per prompt file
- **Include output format** — Show the expected structure of results
- **Add constraints** — Prevent common mistakes and edge cases
- **Use step numbering** — Amazon Q follows numbered steps more reliably
- **Test with real queries** — Verify the prompt produces useful results in practice
