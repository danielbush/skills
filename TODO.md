# TODO

## Up next

- [ ] **Evals for nullables-test** — create fixture files and eval suite for the test skill (similar to nullables-refactor evals)
- [ ] **Incorporate real-world feedback** — human may try skills on another project; adjust based on findings
- [ ] **Big-picture explanation** — skills should be able to explain the whole framework (side-effect boundaries, HARDWIRED vs INJECTED, WRAPPER vs NULLABLE_CLASS, refactor→test pipeline) to reinforce understanding
- [ ] **Improve nullables-refactor skill** — eval gap: missed noting compliant code (e.g. `findUser()` using INJECTED_INFRA). Add guidance to call out what's working, not just violations.
- [ ] **Vocabulary pass (broader)** — `docs/vocabulary.md` exists but is scoped to nullables. Still need: deep modules, on-demand vs ambient skills, the four themes from `specs/discussion/discussion-driven-development.md`.

## Explore next

- [ ] **Generalize @oneput skills** — look at `/Users/danb/projects/@oneput/.agents/skills/remember`, `summarize`, `symbol-lookup` for potential extraction into this repo
- [ ] **Convert jsed to generalized skill** — `/Users/danb/projects/@oneput/.agents/skills/jsed` → new generalized skill in this repo (relates to shared-understanding-skill spec)

## Active

- [ ] **Shared understanding skill** — generalize jsed's vocab + architecture narrative + orientation into a reusable skill; draft `/understand init`, `/understand update`, `/explain <TERM>` → `specs/active/shared-understanding-skill.md`
- [ ] **Design patterns principles** — extract named concepts from nullables algorithm; draft CLAUDE.md principles section; clarify DI/IoC/effect-ts boundary → `specs/active/design-patterns-principles.md`
- [ ] **Generalize specs system** — extract /summarize and /remember from oneput into project-agnostic skills → `specs/active/generalize-specs-system.md`
- [ ] **Project collaboration framework** — define standard file set (AGENTS.md, PRD.md, SESSIONS.md, TODO.md); trial on this repo → `specs/active/project-collaboration-framework.md`

## Done

- [x] Created `nullables-test` skill with precondition checks, instantiation guidance (new vs createNull), and illustrative test philosophy
- [x] Created `nullables-refactor` skill with side-effect boundary classification and eval suite (4 fixtures, 26 assertions)
- [x] Created `docs/vocabulary.md` — 15+ terms including HARDWIRED_INFRA, INJECTED_INFRA, CREATE_BOUNDARY_RULE, NULLABLE_CLASS
- [x] Moved original monolithic nullables skill to `sandbox/nullables-skill/`
- [x] Wrote discussion-driven-development discussion spec (four themes + higher-order motivations)
- [x] Broke discussion into four actionable active specs
- [x] Absorbed old PRD.md and TODO.md into specs system
- [x] Applied collaboration framework to this repo (AGENTS.md, PRD.md, SESSIONS.md, TODO.md)
