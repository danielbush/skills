# TODO

## Up next

- [ ] **Improve nullables-refactor skill** — with-skill eval missed noting compliant code (e.g. `findUser()` using injected `this.db`). Add guidance to call out what's working, not just violations.
- [ ] **Vocabulary pass (broader)** — `docs/vocabulary.md` exists but is scoped to nullables. Still need: deep modules, on-demand vs ambient skills, the four themes from `specs/discussion/discussion-driven-development.md`.
- [ ] **Break up nullables skill** ⚡ — original SKILL.md is still monolithic; extract named concepts, separate into commands (`/nullables test`), move detailed reference material to `references/` → `specs/active/design-patterns-principles.md`

## Active

- [ ] **Shared understanding skill** — generalize jsed's vocab + architecture narrative + orientation into a reusable skill; draft `/understand init`, `/understand update`, `/explain <TERM>` → `specs/active/shared-understanding-skill.md`
- [ ] **Design patterns principles** — extract named concepts from nullables algorithm; draft CLAUDE.md principles section; clarify DI/IoC/effect-ts boundary → `specs/active/design-patterns-principles.md`
- [ ] **Generalize specs system** — extract /summarize and /remember from oneput into project-agnostic skills → `specs/active/generalize-specs-system.md`
- [ ] **Project collaboration framework** — define standard file set (AGENTS.md, PRD.md, SESSIONS.md, TODO.md); trial on this repo → `specs/active/project-collaboration-framework.md`

## Done

- [x] Created `nullables-refactor` skill with side-effect boundary classification and eval suite (4 fixtures, 26 assertions)
- [x] Created `docs/vocabulary.md` — 15 terms scoped to nullables (HARDWIRED_INFRASTRUCTURE, CREATE_BOUNDARY_RULE, NULLABLE_CLASS, etc.)
- [x] Wrote discussion-driven-development discussion spec (four themes + higher-order motivations)
- [x] Broke discussion into four actionable active specs
- [x] Absorbed old PRD.md and TODO.md into specs system
- [x] Applied collaboration framework to this repo (AGENTS.md, PRD.md, SESSIONS.md, TODO.md)
