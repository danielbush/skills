---
status: active
created: 2026-03-17
summary: Define a standard set of project files (AGENTS.md, PRD.md, SESSIONS.md, TODO.md, work/) that together give an agent full continuity across sessions and alignment with the human on goals, principles, progress, and reasoning
parent: ../discussion/discussion-driven-development.md
theme: "Way of Working"
---

## Goal

Define and document a standard project collaboration framework — a small set of files that, together, let a human say "let's continue" and have the agent fully oriented within seconds.

## The artifacts

| File | Purpose | Audience | Cadence |
|---|---|---|---|
| **AGENTS.md** | How to work in this codebase — principles, testing approach, architecture, vocabulary pointers | Agent (always loaded) | Updated when principles change |
| **PRD.md** | Overarching goals and direction — steers both human and agent | Both | Updated when goals shift |
| **SESSIONS.md** | What happened each session — enables "let's continue" | Agent (read on session start) | Appended each session |
| **TODO.md** | Current work items with progress — what's in flight, what's next, what's done | Both | Updated during work |
| **work/** | Institutional memory — why things were done, decisions, reasoning | Both (searched on demand) | Written per session via /summarize |

## How they relate

```
PRD.md          ← where are we going? (goals, direction)
  ↓
TODO.md         ← what are we doing right now? (tasks, progress)
  ↓
SESSIONS.md     ← what did we do? (session log, "let's continue")
  ↓
work/          ← why did we do it? (decisions, reasoning)

AGENTS.md       ← how do we do things here? (principles, always-on)
```

- **PRD.md** is the north star. Human updates it when priorities change. Agent references it to understand whether a task aligns with broader goals.
- **TODO.md** is the current sprint. Tracks what's being worked on, what's blocked, what's done. Agent updates it as work progresses. Human reviews it to steer.
- **SESSIONS.md** is the bridge between sessions. Each entry is brief — date, what was done, what's next. When the human says "let's continue", the agent reads the last entry and picks up.
- **work/** is the deep archive. Not read routinely — searched when the human or agent needs to recall *why* something was done a certain way.
- **AGENTS.md** is the constitution. Doesn't change often. Tells the agent the non-negotiable principles (how to test, how to structure code, where to find vocab/architecture docs).

## Memory model

The artifacts map to different memory time horizons:

| Artifact | Memory type | Lifespan |
|---|---|---|
| Current conversation | Short-term | Resets each session |
| SESSIONS.md | Short-to-mid-term | Bridges sessions, recent history |
| TODO.md | Short-to-mid-term | Current work, changes frequently |
| work/ | Mid-to-long-term | Decisions and reasoning, searchable archive |
| Vocabulary, architecture, principles | Long-term | Durable shared understanding, rarely changes |
| AGENTS.md | Long-term | Constitution, rarely changes |
| PRD.md | Mid-term | Evolves with goals |

This mirrors how human memory works: you remember what you're doing now (context), what you did yesterday (sessions), why you made a decision last month (specs), and the foundational concepts you've internalized (vocabulary/principles).

## Key distinctions

- **SESSIONS.md vs work/**: SESSIONS.md is a *log* — brief, chronological, optimized for "what happened last time." Specs are *documents* — structured, topical, optimized for "why did we decide X."
- **SESSIONS.md vs TODO.md**: SESSIONS.md looks backward (what happened). TODO.md looks forward (what to do next). Together they give full continuity.
- **AGENTS.md vs PRD.md**: AGENTS.md is *how* (principles, conventions). PRD.md is *what* and *why* (goals, priorities). AGENTS.md rarely changes; PRD.md evolves.

## Open questions

1. **SESSIONS.md format** — how much per entry? Propose: date, 2-3 bullet points of what was done, 1-2 bullets of what's next. Should it reference TODO.md items?
2. **Who writes SESSIONS.md?** — the agent at end of session (like /summarize)? The human? Both?
3. **TODO.md format** — simple markdown checklist? Or something more structured with status, priority?
4. **Does this need a skill?** — a `/session-start` skill that reads SESSIONS.md + TODO.md and orients the agent? A `/session-end` skill that appends to SESSIONS.md and updates TODO.md?
5. **Relationship to Theme 3** — the specs system (/summarize, /remember) is one piece of this. Should the collaboration framework be the parent concept, with specs as a component?

## Next steps

1. Draft the format for each file (brief — just enough to be useful)
2. Trial it on this repo (SKILLS_REPO) — write SESSIONS.md and TODO.md here
3. See how it feels across 2-3 sessions before codifying into a skill
