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
