---
created: 2026-03-21
summary: "Vision for session continuity — agent reads three overlapping sources at session start: (1) what happened last session, (2) what was on the human's mind, (3) work progress across recent sessions. .sessions/ handles 1+2, work/ handles 3."
---

# Session Continuity

## The vision

I want to be able to say "let's continue" or "where were we" at the start of each session and the agent looks at:

1. **What happened last session** — what we actually did, discussed, decided
2. **What was on my mind** — concerns, open questions, things bothering me, cross-cutting thoughts that may not belong to any ticket
3. **What we actually did in terms of ticket/work progress** — the factual state of work across recent sessions

(1) and (3) overlap — they're both about what happened. But they differ in scope: (1) is about the *last session specifically*, while (3) might encompass work across several tickets over several recent sessions.

## How the pieces fit

```
.sessions/$USER.md          →  (1) what happened last session
                             →  (2) what was on my mind
                             Short-term bridge, per-user, rotated monthly.
                             Maintains continuity for the human and agent day to day.

work/active/                 →  (3) ticket/work progress across sessions
work/BACKLOG.md                  Each ticket has Status, Changes, Tasks.
                                 Mid-term, shared, evolves across sessions.

docs/                        →  what the project IS
                                 Vocabulary, architecture, themes.
                                 Long-term, ambient, loaded on demand.
```

The session log (`.sessions/`) gives you the *feel* — the human's headspace, what was bothering them, the threads that were live. The work items (`work/`) give you the *facts* — what's done, what's not, where things stand. Together they let the agent orient fully: last session's mood and context + current state of all active work.
