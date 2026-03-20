---
name: work-tracker
description: "Create and manage work items, tickets, and tracking artifacts in a project's work/ directory. Use when the human wants to create a ticket, track work, log a decision, review the backlog, move items between statuses, or scan what's in flight. Triggers on phrases like: 'create a ticket', 'let's track this', 'create a work item', 'what's in the backlog', 'what's active', 'move this to done'. Bootstraps the work/ directory structure on first use if it doesn't exist."
---

# Work Tracker

Create and manage work items in a project's `work/` directory.

## Bootstrap

If `work/` doesn't exist in the project root, create the full structure before doing anything else:

```
work/
  active/       → items being worked on
  done/         → completed items (moved here, not deleted)
  BACKLOG.md    → prioritised queue of future work
```

Create `work/BACKLOG.md` with this header:

```markdown
# Backlog

Items near the top get worked on first. Each item is an initial draft — improve wording and break out into individual files when ready to work on.

Grep-friendly: `grep -A 3 "^## " work/BACKLOG.md` gives a quick index.

---
```

## Work item format

Each work item is a markdown file with frontmatter:

```markdown
---
id: SHORT_UPPER_SNAKE_CASE_ID
status: active | done
created: YYYY-MM-DD
summary: one-line summary
---

# type: Title

Body text — the initial proposal, plan, or discussion.
```

### Fields

- **id** — short, unique, UPPER_SNAKE_CASE. Used for cross-referencing between items (e.g. "see SHARED_UNDERSTANDING"). Keep it descriptive but brief.
- **status** — matches the directory it lives in: `active` or `done`
- **created** — date the item was created
- **summary** — one line, used for scanning

### Title prefix

Use conventional commit style prefixes: `feat:`, `refactor:`, `fix:`, `chore:`, `docs:`. If the type isn't obvious, leave it unclassified.

### File naming

`YYYYMMDD.<type>.<slug>.md`

Examples:
- `20260320.feat.entry-point-questions.md`
- `20260320.chore.explore-pi-harness.md`

In a monorepo with packages, add the package name: `YYYYMMDD.<package>.<type>.<slug>.md`

## Operations

### Create a work item

When the human says "create a ticket", "let's track this", "create a work item", or similar:

1. If it's future work (not something to start now), add it to `work/BACKLOG.md` as an h2 entry with the grep-friendly fields (see below). Otherwise, create a file in `work/active/`.
2. Draft the item with frontmatter, a title, and body from what the human described.
3. If it belongs in the backlog:

```markdown
## type: Title
- id: THE_ID
- drafted: YYYY-MM-DD
- summary: one-line summary

Body text.
```

### Move an item

When the human says "this is done", "promote this to active", or similar:

1. Update the `status:` in frontmatter
2. Move the file to the matching directory (`work/active/` or `work/done/`)

When promoting a backlog item to active:
1. Remove it from `work/BACKLOG.md`
2. Create a file in `work/active/`

### Scan / review

When the human says "what's active", "what's in flight", "show me the backlog", or similar:

- **Active items**: list files in `work/active/`, show id + summary from frontmatter
- **Backlog**: run `grep -A 3 "^## " work/BACKLOG.md` for the quick index
- **All statuses**: scan both directories

### Cross-referencing

Items can reference each other by id (e.g. "relates to SHARED_UNDERSTANDING"). When creating an item that relates to another, mention the id in the body. Don't over-formalise this — a mention is enough.

## Conventions

- Keep items concise. The initial version is a draft — it gets refined as work progresses.
- Don't delete items. Move to `work/done/` when complete.
- The backlog is ordered by priority — items near the top get worked on first.
- When an item is too big, break it into smaller items and link them via id references.
