---
name: odysseus-skill-authoring
description: "Author and manage skills for Odysseus AI Agent at /home/yan/odysseus/data/skills/general/. Covers Odysseus frontmatter format, bulk creation from external sources (GitHub, skills.sh), adapting Hermes skills to Odysseus format, and chapter-based skill splitting."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [odysseus, skills, authoring, adaptation, bulk-creation]
    related_skills: [hermes-agent-skill-authoring]
---

# Odysseus Skill Authoring

## Overview

Odysseus is user Wayan's AI agent framework with its own skill system at `/home/yan/odysseus/data/skills/general/`. Skills use a different frontmatter format than Hermes. This skill covers creating, adapting, and bulk-importing skills into Odysseus.

## When to Use

- User asks to create a new Odysseus skill
- User asks to adapt/port a Hermes skill or external skill (skills.sh, GitHub) to Odysseus
- User wants to bulk-import skills from a GitHub repo (e.g. apollographql/skills)
- User wants to split a multi-chapter skill into individual skills

## Odysseus Skill Format

### Path Convention

```
/home/yan/odysseus/data/skills/general/<skill-name>/SKILL.md
```

- One folder per skill, lowercase with hyphens
- File must be named `SKILL.md` (uppercase, case-sensitive)

### Required Frontmatter (ALL fields mandatory)

```yaml
---
name: skill-name
description: "Description of the skill"
version: 1.0.0
category: general
status: draft
confidence: 0.8
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---
```

| Field | Type | Values | Notes |
|-------|------|--------|-------|
| `name` | string | lowercase, hyphens | Must match folder name |
| `description` | string | quoted | Brief function description |
| `version` | string | semver | `major.minor.patch` |
| `category` | string | `general` | Always `general` for now |
| `status` | string | `draft`, `published`, `archived` | Start as `draft` |
| `confidence` | float | `0.0` - `1.0` | Default `0.8`, raise after proven |
| `source` | string | `learned`, `manual`, `imported` | `learned` for adapted, `imported` for external |
| `owner` | string | `wayan` | Always `wayan` |
| `created` | string | ISO 8601 | Quoted, with `T` and `Z` |

### Key Differences from Hermes Format

| Aspect | Hermes | Odysseus |
|--------|--------|----------|
| Path | `~/.hermes/skills/<category>/<name>/SKILL.md` | `/home/yan/odysseus/data/skills/general/<name>/SKILL.md` |
| Required fields | `name`, `description` only | 9 fields ALL required |
| Extra fields | `author`, `license`, `metadata.hermes.tags` | `status`, `confidence`, `source`, `owner`, `created` |
| Category | In folder path | In frontmatter `category` field |
| Creation tool | `skill_manage(action='create')` | `write_file` directly |

## Workflow: Adapting a Hermes Skill

1. Load the Hermes skill with `skill_view(name='skill-name')`
2. Extract the content body (everything after frontmatter)
3. Replace frontmatter with Odysseus format (all 9 fields)
4. Remove Hermes-specific references (tool names like `skill_view`, `skill_manage`, `delegate_task` etc.) unless Odysseus has equivalents
5. Write to `/home/yan/odysseus/data/skills/general/<name>/SKILL.md`

## Workflow: Importing from GitHub/skills.sh

1. Fetch raw content: `curl -sL "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/skills/<name>/SKILL.md"`
2. If skill has `references/` chapters, fetch each: `curl -sL ".../references/chapter_XX.md"`
3. For chapter-based skills, create **one index skill** + **one skill per chapter**
4. Each gets its own folder with Odysseus frontmatter
5. Index skill lists all chapters with their skill names in a table

### Bulk Chapter Creation Pattern

When a source skill references multiple chapters/references:

```python
# Fetch all chapters in a loop
for i in range(1, N+1):
    content = curl(f".../references/chapter_{i:02d}.md")
    # Wrap with Odysseus frontmatter
    # Write to /home/yan/odysseus/data/skills/general/<slug>/SKILL.md
```

Naming convention for chapters:
- Index: `rust-best-practices` (overview + chapter table)
- Chapters: `rust-coding-styles-idioms`, `rust-error-handling`, etc.
- Pattern: `<topic>-<chapter-subject>` — descriptive, not numbered

## Workflow: Creating a "Global" Variant

When user wants a universal version of a stack-specific skill (e.g. Tauri PRD → Global PRD):

1. Take the specific skill as template
2. Remove all stack-specific references
3. Add tech stack selection as first clarifying question
4. Add redirect rule: if user picks the specific stack → redirect to specific skill
5. Generalize mandatory standards to be stack-agnostic
6. Add multiple folder structure examples for common stacks

## Common Pitfalls

1. **Missing frontmatter fields.** Odysseus requires ALL 9 fields. Hermes only requires 2. Always include `status`, `confidence`, `source`, `owner`, `created`.
2. **Wrong path.** Odysseus skills go to `/home/yan/odysseus/data/skills/general/`, not `~/.hermes/skills/`.
3. **Using `skill_manage`.** That writes to Hermes skill tree. Use `write_file` for Odysseus.
4. **Folder name ≠ frontmatter name.** These must match exactly.
5. **Forgetting source attribution.** When importing external skills, add a source note at the bottom with link to original repo.
6. **Chapter skills without index.** Always create an index/overview skill that lists all chapters with their skill names.
7. **Not checking for existing skills.** Run `ls /home/yan/odysseus/data/skills/general/` first to avoid overwriting or duplicating.

## Verification Checklist

- [ ] File at correct path: `/home/yan/odysseus/data/skills/general/<name>/SKILL.md`
- [ ] All 9 frontmatter fields present
- [ ] `name` matches folder name
- [ ] `description` is quoted and informative
- [ ] `owner: wayan` and `created` timestamp set
- [ ] Body content is non-empty after frontmatter
- [ ] Source attribution included for imported skills
- [ ] No Hermes-specific tool references unless Odysseus supports them
- [ ] Index skill created for chapter-based imports
