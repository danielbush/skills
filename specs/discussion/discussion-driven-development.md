---
status: discussion
created: 2026-03-17
summary: Three-theme framework for human-agent collaboration — shared understanding (vocab + architecture narrative), design patterns (nullables, deep modules, DI/IoC), and institutional memory (specs, remember, summarize) — connected by the insight that humans and agents both work well at high compression
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

## Open Questions

1. **Generalizability of Theme 1** — the jsed vocab/architecture/orientation works because it was built by hand over time. Can a skill bootstrap this for a new codebase? Or is the value in the *process* (human + agent build it together session by session) rather than automation?

2. **Where do the design patterns live?** Options:
   - All in one "architecture" skill
   - Nullables stays separate (it has a concrete workflow); the rest go in CLAUDE.md as principles
   - Each pattern gets its own skill (probably overkill)

3. **Are the three themes one system or three?** The name "discussion-driven development" implies one system. But the implementation might be three independent skills that happen to share vocabulary conventions.

4. **Spec system scope** — `/summarize` and `/remember` are currently in the oneput repo. Are they general-purpose? Could they work across projects?

5. **DI vs IoC boundary** — where does dependency injection end and inversion of control begin? How does effect-ts fit? This needs clarifying before it can be encoded as a principle or skill.
