---
status: discussion
created: 2026-03-17
summary: Four-theme framework for human-agent collaboration — shared understanding, design patterns, institutional memory, and a project collaboration framework (AGENTS.md, PRD.md, SESSIONS.md, TODO.md) — connected by the insight that humans and agents both work well at high compression
---

# Discussion-Driven Development

## The Meta-Idea

Humans and agents both work well at high compression. A human says "make this nullable" or "SPLIT_BY_TOKEN after the CURSOR" and the agent can unpack that into dozens of concrete code changes. But this only works if:

1. **There's shared vocabulary** — so compressed statements are unambiguous
2. **There are encoded patterns** — so the agent knows *how* to unpack
3. **There's memory of why** — so neither human nor agent makes decisions that contradict past reasoning

These three form a system:

```
Vocabulary ──→ enables compressed communication
Patterns   ──→ enable compressed instructions ("make it nullable")
Memory     ──→ preserves the reasoning that vocabulary and patterns can't capture
```

## Higher-Order Motivations

### Keeping the human technically engaged

As agents take on more implementation work, the human's role shifts from writing code to steering direction. But there's a risk: the human drifts away from the technical reality and loses the ability to make good decisions. The vocabulary and discussion-driven approach addresses this — it keeps the human engaged at the *conceptual* level even when they're not writing code. Naming things, reviewing architecture narratives, discussing trade-offs in shared terms — these are all ways the human stays across what's happening without needing to read every line of code. This extends beyond a single codebase: a human orchestrating multiple systems needs to hold the essential structure of each in their head, and shared vocabulary is how you compress a system down to something a human can carry.

### Beyond markdown files: toward a graph of managed systems

The current implementation (md files in a repo) is a starting point, not the end state. The artifacts described here — vocabulary, principles, session history, decisions, goals — are really **nodes and relationships in a graph**. A vocabulary term is used in an architecture narrative, which is referenced by a spec, which was produced in a session, which advanced a goal in a PRD. Today these relationships are implicit (markdown links, `parent:` frontmatter). Eventually this could be a server-backed graph where:

- Vocabulary, principles, decisions, and goals are nodes
- Relationships (uses, depends-on, produced-by, supersedes) are edges
- Multiple codebases and systems connect to the same graph
- An agent working on system A can see that a decision in system B affects it
- The human gets a unified view across all the systems they manage

The md-files-in-a-repo approach is the right starting point — it's simple, versionable, and you can feel out the right structures before committing to infrastructure. But the design should keep this future in mind: the artifacts should be structured enough that migrating to a graph later is straightforward.

---

## Theme 1: Shared Understanding of a Codebase

### What exists (jsed as the reference implementation)

- **Vocabulary** (`docs/vocabulary.md`) — named domain concepts in UPPER_SNAKE_CASE (TOKEN, FOCUSABLE, LINE_SIBLING...). Human says "SPLIT_BY_TOKEN after the focused TOKEN" and the agent knows exactly what to do.
- **Architecture narrative** (`docs/architecture.md`) — a story that builds bottom-up, each section depending only on what came before. Designed so a human can read top-to-bottom and understand the whole system before touching code.
- **Deep modules structure** — top-level files show *what*, `lib/` shows *how*. The architecture narrative maps to the top level; you only dive into `lib/` to change internals.
- **Orientation skill** (`/jsed`) — reads the above in order and summarizes, so a new session starts with shared context.

### The generalization question

Any codebase could have this: vocab + narrative + orientation skill. The question is whether a skill can *bootstrap* these artifacts for an arbitrary codebase, or whether the value is in the *process* (human + agent build it together session by session).

### Reference implementations

- jsed AGENTS.md: `/Users/danb/projects/@oneput/packages/jsed/AGENTS.md`
- jsed architecture: `/Users/danb/projects/@oneput/packages/jsed/docs/architecture.md`
- jsed vocabulary: `/Users/danb/projects/@oneput/packages/jsed/docs/vocabulary.md`
- jsed orientation skill: `/Users/danb/projects/@oneput/.agents/skills/jsed/SKILL.md`

---

## Theme 2: Design Patterns for Testable, Changeable Code

### What exists

- **Nullables skill** (`skills/nullables/SKILL.md`) — the full algorithm for `.create`/`.createNull`, infrastructure wrappers, embedded stubs, configurable responses, output trackers, cascading nullables, delayed instantiation.
- **PRD.md** — identifies the broader family: A-Frame architecture, deep modules, consumer-defined interfaces, value objects, error mapping (neverthrow), state-based testing.

### What these have in common

They're all about making code **easier to test** (no mocks, fast, reliable) and **safer to change** (clear boundaries, dependency injection/inversion, deep modules absorbing complexity). They're *prescriptive* — "when you write code, do it this way."

### What's not yet crystallized

- Deep modules, DI vs IoC, consumer-defined interfaces — these are principles in PRD.md but don't have their own skill or actionable workflow yet.
- The boundary between "always-on principle" (CLAUDE.md) and "on-demand workflow" (skill) needs deciding per concept.

---

## Theme 3: Institutional Memory (Why Things Were Done)

### What exists (oneput as the reference implementation)

- **Spec files** (`specs/done/`, `specs/active/`, `specs/discussion/`) — session summaries with frontmatter, organized by status. Each captures *changes* and *decisions* (especially the *why*).
- **/summarize** skill — writes a spec at end of session, capturing what was done and why.
- **/remember** skill — searches specs to recall past work, decisions, context.

### What this solves

Agent context resets between sessions. Specs are the persistent memory of *reasoning* — not what the code looks like (that's in the code), but *why* it looks that way. Vocabulary terms in specs connect them to the shared understanding from Theme 1.

### Reference implementations

- Spec files: `/Users/danb/projects/@oneput/specs/`
- Summarize skill: `/Users/danb/projects/@oneput/.agents/skills/summarize/SKILL.md`
- Remember skill: `/Users/danb/projects/@oneput/.agents/skills/remember/SKILL.md`

---

## Theme 4: Project Collaboration Framework (Way of Working)

Beyond the specs system, there's a broader set of project files that together give the agent full continuity:

| File | Purpose |
|---|---|
| **AGENTS.md** | How to work here — principles, testing, architecture, vocab pointers (always loaded) |
| **PRD.md** | Where are we going — overarching goals that steer both human and agent |
| **SESSIONS.md** | What happened — session log, enables "let's continue" |
| **TODO.md** | What's in flight — current tasks with progress |
| **specs/** | Why we did it — decisions and reasoning (searched on demand) |

The key insight: these files serve different time horizons. AGENTS.md rarely changes (constitution). PRD.md evolves with goals. TODO.md changes daily. SESSIONS.md is append-only. Specs are the permanent archive. Together they let a human start a session with "let's continue" and the agent is fully oriented.

See `active/project-collaboration-framework.md` for the detailed breakdown.

---

## Open Questions

1. **Generalizability of Theme 1** — the jsed vocab/architecture/orientation works because it was built by hand over time. Can a skill bootstrap this for a new codebase? Or is the value in the *process* (human + agent build it together session by session) rather than automation?

2. **Where do the design patterns live?** Options:
   - All in one "architecture" skill
   - Nullables stays separate (it has a concrete workflow); the rest go in CLAUDE.md as principles
   - Each pattern gets its own skill (probably overkill)

3. **Are the three themes one system or three?** The name "discussion-driven development" implies one system. But the implementation might be three independent skills that happen to share vocabulary conventions.

4. **Spec system scope** — `/summarize` and `/remember` are currently in the oneput repo. Are they general-purpose? Could they work across projects?

5. **DI vs IoC boundary** — where does dependency injection end and inversion of control begin? How does effect-ts fit? This needs clarifying before it can be encoded as a principle or skill.
