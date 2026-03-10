---
name: udemy-scanner
description: >
  This skill should be used when the user asks to "search Udemy", "find a Udemy course",
  "get course details from Udemy", "scan Udemy", "research a Udemy course", "summarise a Udemy course",
  "compare Udemy courses", or provides a Udemy URL and wants course information extracted.
  Also triggers when the user mentions Udemy-related course research or training recommendations.
version: 0.1.0
---

# Udemy Course Scanner

Scan, search, and extract structured information from Udemy courses using a multi-strategy
retrieval approach that works around Udemy's direct-fetch restrictions.

## Important: Udemy Blocks Direct Fetching

Udemy returns **403 Forbidden** on direct `WebFetch` requests. Never attempt to fetch
`udemy.com` URLs directly — it will always fail. Use the retrieval strategies below instead.

## Retrieval Strategies (in priority order)

### Strategy 1: Class Central Mirror

Class Central indexes Udemy courses with structured metadata. To find a course:

1. Extract the course slug from the Udemy URL (e.g., `python-for-network-engineers` from
   `https://www.udemy.com/course/python-for-network-engineers/`)
2. Search: `site:classcentral.com udemy "<course-slug-keywords>"`
3. Fetch the Class Central page — it typically includes: instructor, duration, rating,
   student count, description, and curriculum outline

**Class Central URL pattern:**
`https://www.classcentral.com/course/udemy-<course-slug>-<id>`

### Strategy 2: WebSearch Aggregation

Run multiple targeted searches in parallel to triangulate course details:

```
Search 1: udemy "<exact course title>" instructor hours lectures rating
Search 2: "<exact course title>" udemy review curriculum topics
Search 3: site:classcentral.com OR site:coursetalk.com "<course title>" udemy
```

Extract and cross-reference details from search snippets. Review aggregator sites
(Medium "best courses" articles, dev.to recommendations) often include hours,
lecture counts, and ratings inline.

### Strategy 3: Instructor Discovery + Deep Search

If the course slug contains an instructor subdomain (e.g., `marcelclasses.udemy.com`),
search specifically for that instructor's courses:

```
"<instructor-name>" udemy course "<course-title-keywords>"
```

Instructor personal sites and LinkedIn profiles often contain complete course details.

## Output Format

Structure all course information in this format:

```markdown
## Course Title

| Field | Detail |
|---|---|
| **Platform** | Udemy |
| **Instructor** | Name |
| **Duration** | X hours |
| **Lectures** | N lectures |
| **Rating** | X.X / 5 (N ratings) |
| **Students** | N enrolled |
| **Last Updated** | Month Year |
| **Level** | Beginner / Intermediate / Advanced |
| **URL** | [Link](url) |

### Description
Brief course summary.

### Key Topics
- Topic 1
- Topic 2

### What You'll Learn
- Learning outcome 1
- Learning outcome 2

### Prerequisites
- Prerequisite 1
```

Mark any field as `Not available` if it cannot be confirmed from search results.
Never fabricate data — accuracy over completeness.

## Workflow

### Single Course Lookup

1. Identify the course URL or title
2. Extract the course slug
3. Run Strategy 1 (Class Central) and Strategy 2 (WebSearch) **in parallel**
4. If instructor is unknown, run Strategy 3
5. Merge results, resolve conflicts by preferring Class Central data
6. Output in the structured format above

### Course Comparison

When comparing multiple courses:

1. Run parallel lookups for all courses simultaneously
2. Present results in a comparison table:

```markdown
| Feature | Course A | Course B | Course C |
|---|---|---|---|
| Duration | X hrs | Y hrs | Z hrs |
| Rating | X.X | Y.Y | Z.Z |
| Key Focus | ... | ... | ... |
```

3. Include a recommendation section tailored to the user's stated role or goals

### Bulk Search (Topic Discovery)

When searching for courses on a topic:

1. Search: `udemy best courses "<topic>" <current-year> review`
2. Search: `site:classcentral.com udemy "<topic>"`
3. Collect top 5-10 results
4. Run single-course lookups on the top candidates in parallel
5. Present as a ranked list with key differentiators

## Handling Failures

- **403 from WebFetch**: Expected. Do not retry. Fall back to WebSearch strategies.
- **No Class Central match**: Rely on WebSearch aggregation; note reduced confidence.
- **Missing fields**: Mark as `Not available` — never guess or interpolate.
- **Stale data**: Prefer results from the current and previous year. Flag if the most recent source is older than 18 months.

## Additional Resources

### Reference Files

- **`references/search-patterns.md`** — Pre-built search query templates for common
  course categories (networking, AI/ML, DevOps, cloud, programming languages)
