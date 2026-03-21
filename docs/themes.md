---
id: KNOWLEDGE_LAYERS
status: discussion
created: 2026-03-20
summary: "Three knowledge layers: world-building (vocab, architecture — ambient, project-bound, in docs/), skills (portable, activated, project-independent), strategic thinking (the sharpening process that produces the other two, also in docs/)."
---

# chore: Knowledge layers

The goal: overlapping memory and context management to create agents that know what you were doing recently and last time, what was on your mind, what was bothering you, and what the project is about and how it works.

These layers are how that's achieved.

## The layers

### World-building (ambient, long-term, project-bound)

Vocabulary, architecture narrative, entry-point questions. This is institutional memory tied to a project, an entity. It's ambient — it evolves with the project and just the right parts are loaded into short-term memory (context) when needed to create continuity and maintain the "world."

Lives *in* the project: `docs/vocab/`, `docs/architecture.md`, AGENTS.md. Not portable — it *is* the project's identity.

### Skills (portable, activated, project-independent)

Carry knowledge about *how to do things* across projects. Activated at a given time, applied to different projects. Portable memory — the work-tracker, nullables-refactor, etc.

Live in this repo, installed into projects. The knowledge travels with the skill.

### Strategic thinking (the sharpening)

Not a ticket. Not a deliverable. The ongoing conversation between human and agent about *what are we actually doing and why*. This is where both sides sharpen their understanding over time.

Strategic thinking *produces* vocabulary, architecture, and principles as outputs — but itself isn't any of those things. It's the process that leads to world-building artifacts crystallising.

Examples: the four central themes, the meta-idea about compression, the higher-order motivations. These emerged from discussion, not from task work.

## Where does each live?

| Layer | Location | Character |
|---|---|---|
| World-building | `docs/`, AGENTS.md | Settled, ambient, loaded on demand |
| Skills | `skills/` (this repo) | Portable, installed into projects |
| Strategic thinking | `docs/` | Evolving, sharpens over time, long-term |
| Backlog | `work/BACKLOG.md` | Quick sketches, prioritised future work |
| Active work | `work/active/` | In progress, being delivered |

## Open questions

- **Resolved**: strategic thinking lives in `docs/` alongside vocabulary and other long-term artifacts. `work/` is for actionable items only (active work + backlog).
- How does strategic thinking relate to SESSIONS.md? Sessions capture *what happened*; docs capture *what we're figuring out* and *what we've figured out*. They're complementary — a session might advance a theme doc.
- Themes and strategic docs evolve in place — they don't move between directories like work items do.
