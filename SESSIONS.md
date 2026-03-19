# Sessions

## 2026-03-17

### What happened

- Started from rough notes in `specs/active/discussion-driven-development.md` about vocab, architecture, and nullable skills
- Unpacked into four themes: shared understanding, design patterns, institutional memory, project collaboration framework
- Identified higher-order motivations: keeping the human technically engaged, grounding through examples, the graph future, vocabulary as agent-internal language that surfaces when helpful (configurable to always-on)
- Wrote discussion spec (`specs/discussion/discussion-driven-development.md`) capturing the full picture
- Broke into four actionable specs in `specs/active/`
- Absorbed old PRD.md → `specs/discussion/architecture-skills-prd.md`, old TODO.md into design-patterns spec
- Applied the collaboration framework to this repo itself: AGENTS.md, PRD.md, TODO.md, SESSIONS.md
- Noted memory model: context (short) → SESSIONS.md (short-to-mid) → specs (mid-to-long) → vocab/architecture/principles (long-term)

### What's next

1. **Vocabulary pass** — name the key principles and entities across the discussion spec: deep modules, nullables, on-demand vs ambient skills, the themes themselves. Start building `docs/vocabulary.md` for this repo.
2. **Break up nullables skill** — the current SKILL.md is monolithic (332 lines). Make it actionable: extract named concepts, separate into commands (`/nullables refactor`, `/nullables test`, etc.), move detailed reference to `references/`. This is a priority.
3. Then: specs system, shared understanding skill, design patterns principles.

## 2026-03-18

### What happened

- Created `nullables-refactor` skill (`skills/nullables-refactor/SKILL.md`) — analyzes a file and produces a refactoring plan for the nullables pattern
- Built `docs/vocabulary.md` with 15 named terms grounded in the skill's concepts
- Reworked the classification model: replaced A_FRAME layers with **side-effect boundaries** (PURE, IN_MEMORY, OUTSIDE_WORLD) — the nullables machinery targets OUTSIDE_WORLD specifically
- Named key concepts through discussion:
  - **HARDWIRED_INFRASTRUCTURE** — the anti-pattern (I/O imported and used directly, not injected)
  - **CREATE_BOUNDARY_RULE** — dependency `.create()` calls must be lexically inside the class's static `.create()`
  - **NULLABLE_CLASS** — a class with DUAL_FACTORY that receives INFRASTRUCTURE_WRAPPERs via injection (vs an INFRASTRUCTURE_WRAPPER which *contains* the I/O directly)
- Added the critical first-question in the skill: "Is this an INFRASTRUCTURE_WRAPPER or a NULLABLE_CLASS?"
- Created eval suite with 4 fixture files testing: HARDWIRED_INFRASTRUCTURE functions, mixed class, CREATE_BOUNDARY_RULE violations, and already-compliant code
- Ran evals using skill-creator: with-skill 95.8% vs without-skill 88.5% — skill's value is in vocabulary enforcement, structured output, and the WRAPPER vs NULLABLE_CLASS decision
- Added `task evals:nullables-refactor` to Taskfile.yml
- Updated README.md with both nullable skills and eval docs

- Later in session:
  - Created `nullables-test` skill — writes illustrative tests after refactoring, with precondition checks (recursive nullability verification)
  - Clarified test instantiation: `new Foo(Bar.createNull({...}))` when testing Foo itself (fine-grained CONFIGURABLE_RESPONSE control) vs `Foo.createNull()` when Foo is a dependency
  - Renamed HARDWIRED_INFRASTRUCTURE → **HARDWIRED_INFRA**, added **INJECTED_INFRA** (the opposite — properly extracted and injected through CREATE_BOUNDARY_RULE)
  - Moved original monolithic nullables skill to `sandbox/nullables-skill/`
  - Refactor skill now points to test skill as next step

### Continued (later)

- Fixed vocabulary sharing: `docs/vocabulary.md` is the single source, symlinked from `skills/nullables-refactor/references/vocabulary.md` and `skills/nullables-test/references/vocabulary.md`. Both `pnpx skills` and `bunx skills add` resolve the symlinks, so target projects get the real file.
- Updated SKILL.md in both skills to reference `references/vocabulary.md` instead of `docs/vocabulary.md`.
- Added @oneput skills pointers to TODO.md ("Explore next" section).

### What's next

1. **Evals for nullables-test** — create fixture files and eval suite for the test skill, similar to what we did for nullables-refactor
2. **Feedback from real usage** — human may try skills on another project; incorporate feedback
3. **Big-picture explanation** — the agent should be able to explain the whole framework (side-effect boundaries, HARDWIRED vs INJECTED, WRAPPER vs NULLABLE_CLASS, the refactor→test pipeline) so humans get it reinforced and can adjust
4. **Improve refactor skill** — eval gap: missed noting compliant code (e.g., `findUser()` correctly uses injected `this.db`)
5. **Vocabulary pass (broader)** — current vocabulary is scoped to nullables. Still need: deep modules, on-demand vs ambient skills, the four themes
6. **Explore @oneput skills** — look at `remember`, `summarize`, `symbol-lookup` at `/Users/danb/projects/@oneput/.agents/skills/` for potential generalization into this repo; plan how to convert `jsed` to a new generalized skill here
7. Then: shared understanding skill, design patterns principles, specs system

## 2026-03-20

### What happened

- Captured testing principles: AAA pattern (explicit act step), prefer middle/top-down over exhaustive bottom-up, `new` for thing under test vs `createNull` for dependencies (confirmed per James Shore)
- Discussed `/explain-nullables` as a standalone skill; added to backlog
- Discussed entry-point questions — FAQ-style "doors into" a system on top of vocabulary + architecture narrative
- Added mcporter skill, opensrc skill, spec scanner skill, pi harness exploration to backlog
- Discussed AGENTS.md routing for automatic orientation (hardwire into per-package AGENTS.md instead of requiring skill invocation)
- Created `specs/BACKLOG.md` with grep-friendly format (`grep -A 3 "^## "` for quick index)
- Consolidated all nullables work into single active spec `specs/active/nullables-skills.md`
- Removed TODO.md; active work now lives in `specs/active/`, backlog in `specs/BACKLOG.md`
- Reworked "Continuing a session" → "Starting a session" in CLAUDE.md — agent now prompts human with 3 options
- Merged PRD.md content into discussion spec; PRD.md is now a 3-line pointer

### For next time

**Primary concern: portability and consolidation.** Two related worries:

1. **Portability** — how to make the spec system, vocabulary, orientation, and other tooling portable so it can be used across projects (not just this repo or @oneput)
2. **Consolidation** — incorporate learnings from `/Users/danb/projects/@oneput/packages/jsed` and the initial skill attempts in `/Users/danb/projects/@oneput/.agents/skills` back into this repo as the single source, while making it easy to update and consume over in @oneput and its sister packages

These cut across several backlog items (SHARED_UNDERSTANDING, GENERALIZE_SPECS, GENERALIZE_ONEPUT, GENERALIZE_JSED) but the headline is: **this repo should be the source of truth for skills and systems, with a smooth path to use them in other projects.**
