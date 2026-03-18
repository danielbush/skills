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

### What's next

1. **Improve the skill based on eval gap** — the with-skill run missed noting compliant code (e.g., `findUser()` correctly uses injected `this.db`). Add guidance to call out what's working, not just violations.
2. **Vocabulary pass (broader)** — current vocabulary is scoped to nullables-refactor. Still need to name broader concepts: deep modules, on-demand vs ambient skills, the four themes.
3. **Break up nullables skill** — the original SKILL.md is still monolithic. The refactor skill is now separate, but the main skill still needs splitting.
4. Then: shared understanding skill, design patterns principles, specs system.
