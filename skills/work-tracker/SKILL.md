---
name: work-tracker
description: "Create and manage work items, tickets, and tracking artifacts in a project's work/ directory. Also summarises sessions and searches past work. Use when the human wants to create a ticket, track work, log a decision, review the backlog, move items between statuses, scan what's in flight, summarise what was done in a session, or recall past work and decisions. Triggers on phrases like: 'create a ticket', 'let's track this', 'create a work item', 'what's in the backlog', 'what's active', 'move this to done', 'summarise this session', 'write up what we did', 'remember when we...', 'what did we do with...'. Bootstraps the work/ directory structure on first use if it doesn't exist."
---

# Work Tracker

Create and manage work items in a project's `work/` directory. Also summarises sessions and searches past work. Sits on top of the project's vocabulary and architecture (in `docs/`) — work-tracker handles what you were doing recently and last time, what was on your mind, and what was bothering you. The vocabulary and architecture layer handles what the project is about and how it works.

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
id: SHORT_UPPER_SNAKE_CASE__WORK
status: active | done
created: YYYY-MM-DD
summary: one-line summary
---

# type: Title

Body text — the initial proposal, plan, or discussion.

## Status

Where we're at right now. Update this as work progresses — it should reflect the current state at a glance. Recent progress may be implied by items marked off below.

## Tasks

- [ ] First thing to do
- [ ] Second thing to do
- [x] Already done

## Housekeeping

Optional. Things to do in addition to the main work — cleanup, follow-ups, related chores that don't warrant their own ticket.
```

### Fields

- **id** — short, unique, UPPER_SNAKE_CASE with `__WORK` suffix (double underscore to visually separate). Used for cross-referencing between items (e.g. "see SHARED_UNDERSTANDING__WORK"). Keep it descriptive but brief.
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

Items can reference each other by id (e.g. "relates to SHARED_UNDERSTANDING__WORK"). When creating an item that relates to another, mention the id in the body. Don't over-formalise this — a mention is enough.

### Summarise a session

When the human says "summarise this session", "write up what we did", "capture this", or at the end of a session:

1. Create or update a work item in `work/active/` for the session's work. Use the naming convention: `YYYYMMDD.<type>.<topic>.md`.
2. If a file already exists for this session (same date and topic), update it rather than creating a new one. Append new changes and decisions, update the summary. Don't duplicate content already captured.
3. Structure the body with two sections:

**Changes** — focus on conceptual changes, not implementation details. Group by theme, not by file. Use vocabulary terms (UPPER_SNAKE_CASE) from the project's `docs/vocabulary.md` when they apply.

**Decisions** — record design decisions and the *why* behind them. These are the things that would be lost between sessions: motivation, rejected alternatives, non-obvious reasoning.

4. The frontmatter `summary:` is the most important line — optimise it for scanning by a future reader (human or agent) skimming `work/` to find relevant context.
5. Keep it concise — a summary should be shorter than the conversation that produced it. Focus on decisions and the why; the what is in the code.

### Search past work

When the human says "remember when we...", "what did we do with...", or asks about previous sessions, past decisions, or earlier implementations:

1. Glob for files across `work/done/`, `work/active/`, and `work/BACKLOG.md`.
2. Match the query against filenames first (fast path) — the filename scheme encodes type and topic.
3. If filename matches are found, read their frontmatter summaries. Present matches with status, date, and summary so the human can pick which to dive into.
4. If no filename matches, grep the files for the query term in their content, including vocabulary terms (UPPER_SNAKE_CASE).
5. Only read the full body if the human asks to go deeper. If multiple matches, list them and let the human choose.

## Conventions

- Keep items concise. The initial version is a draft — it gets refined as work progresses.
- Don't delete items. Move to `work/done/` when complete.
- The backlog is ordered by priority — items near the top get worked on first.
- When an item is too big, break it into smaller items and link them via id references.
