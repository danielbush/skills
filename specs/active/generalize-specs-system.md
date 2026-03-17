---
status: active
created: 2026-03-17
summary: Extract the specs/summarize/remember system from oneput into a general-purpose skill set that works across projects
parent: ../discussion/discussion-driven-development.md
theme: "Theme 3: Institutional Memory"
---

## Goal

Make the spec-based memory system (summarize + remember) available to any project, not just oneput.

## What exists today (in oneput)

- `specs/` directory with `done/`, `active/`, `discussion/` subdirectories
- `/summarize` — captures session work as a spec file with frontmatter (status, date, summary) + Changes + Decisions sections
- `/remember` — searches specs by filename and content to recall past work

## What needs to change for generalization

1. **Decouple from oneput paths** — currently hardcoded to `specs/` relative to oneput root. Should work with any project that has a `specs/` directory (or configurable location).
2. **Vocabulary awareness** — `/summarize` references jsed vocabulary. Generalized version should use whatever vocabulary exists in the project's `docs/vocabulary.md` (ties back to Theme 1).
3. **Spec naming convention** — `YYYYMMDD.<package>.<topic>.md` assumes a monorepo with packages. For single-package repos, the package segment could be the project name or omitted.

## Next steps

1. Copy `/summarize` and `/remember` skills into this repo (SKILLS_REPO)
2. Parameterize the paths (specs dir, vocabulary file)
3. Test on this repo itself — use specs/ here as the test case
