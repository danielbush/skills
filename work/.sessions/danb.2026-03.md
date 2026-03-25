## 2026-03-17

### What happened
- Unpacked rough notes into four themes: shared understanding, design patterns, institutional memory, project collaboration framework
- Identified higher-order motivations: keeping the human technically engaged, grounding through examples, vocabulary as agent-internal language
- Wrote `docs/discussion-driven-development.md` capturing the full picture
- Broke into four actionable work items in `work/active/`
- Applied the collaboration framework to this repo itself

### On my mind
- How to name the key principles and entities — vocabulary pass needed
- The nullables skill is monolithic (332 lines) and needs breaking up — this is a priority

### Open threads
- NULLABLES_SKILLS__WORK — monolithic skill needs splitting
- SHARED_UNDERSTANDING__WORK, DESIGN_PATTERNS__WORK, GENERALIZE_SPECS__WORK, COLLAB_FRAMEWORK__WORK — all freshly created

## 2026-03-18

### What happened
- Created `nullables-refactor` skill with side-effect boundary classification (PURE, IN_MEMORY, OUTSIDE_WORLD)
- Named key concepts: HARDWIRED_INFRA, CREATE_BOUNDARY_RULE, NULLABLE_CLASS — the first question is "INFRASTRUCTURE_WRAPPER or NULLABLE_CLASS?"
- Ran evals: with-skill 95.8% vs without-skill 88.5%
- Created `nullables-test` skill with precondition checks and illustrative test philosophy
- Built `docs/vocabulary.md` with 15+ terms
- Fixed vocabulary sharing via symlinks

### On my mind
- Eval gap: refactor skill misses noting compliant code — should call out what's working, not just violations
- Need evals for the test skill too
- Vocabulary is nullables-scoped — broader pass still needed (deep modules, on-demand vs ambient skills, the four themes)
- Want to explore @oneput skills (remember, summarize, symbol-lookup, jsed) for generalisation

### Open threads
- NULLABLES_SKILLS__WORK — both skills created, evals for test skill outstanding, refactor eval gap
- VOCAB_BROADER__WORK — vocabulary needs extending beyond nullables

## 2026-03-20

### What happened
- Captured testing principles: AAA (explicit act step), prefer middle/top-down, `new` for thing under test (per James Shore)
- Discussed `/explain-nullables`, entry-point questions, mcporter, opensrc, pi harness — all added to backlog
- Created `work/BACKLOG.md` with grep-friendly format
- Consolidated nullables work into single active work item
- Removed TODO.md and PRD.md — active work in `work/active/`, backlog in `work/BACKLOG.md`, themes in `docs/`
- Renamed `specs/` → `work/`, moved discussion docs to `docs/`
- Created work-tracker skill with session continuity, summarisation, and search
- Discussed knowledge layers: world-building (docs/) vs skills (portable) vs strategic thinking (docs/)
- Added architecture narrative to top of backlog
- Folded remember/summarize capabilities into work-tracker

### On my mind
- **Portability is the headline concern**: how to make vocab, architecture, work system portable across projects — this repo should be the source of truth
- **Consolidation**: incorporate learnings from jsed and @oneput skills back here, make it easy to consume from other projects
- Work-tracker skill is a first draft — needs validation on a real project
- The "on my mind" section is the thing that work items don't capture — the human's headspace
- Skill triggering is semantic (LLM judgment) — description quality matters for reliable activation

### Open threads
- WORK_TRACKER__WORK — first draft done, needs real-world testing
- NULLABLES_SKILLS__WORK — evals for test skill, refactor eval gap, testing principles to incorporate
- ARCHITECTURE_NARRATIVE__WORK — top of backlog, port jsed story-telling approach
- GENERALIZE_ONEPUT__WORK, GENERALIZE_JSED__WORK — extract and consolidate skills from @oneput

## 2026-03-25

### What happened
- Added test-level guideline to nullables-test skill Philosophy section: "build confidence from the top down" — because nullables make sociable tests practical at higher levels, start there and drop down as needed. Not anti-lower-level, just pragmatic.
- Ticked off test level and instantiation on NULLABLES_SKILLS__WORK — AAA pattern is the remaining testing principle to incorporate.
- Extended work-tracker skill with unsupervised ticket support: `backlog/unsupervised/` → `active/unsupervised/`, with `mode: unsupervised` frontmatter, self-contained instructions, and acceptance criteria. `active/` stays the human+model space.
- Attempted to add jcodemunch-mcp as MCP server — added to `.mcp.json` but couldn't connect mid-session. Needs a session restart.

### On my mind
- **Unsupervised work** — just added the structure, want to try it out and see if the self-contained instructions are practical to write and sufficient for an agent to run with
- **jcodemunch-mcp** — interested in building a skill that uses `bunx mcporter` to query it for code search. Need to see what tools it exposes first (restart session, then `mcporter list` or inspect directly)
- The backlog item JDOCMUNCH__WORK references "jdocmunch-mcp" but the actual package is `jcodemunch-mcp` — may need updating

### Open threads
- NULLABLES_SKILLS__WORK — AAA pattern still to incorporate; evals for test skill outstanding
- WORK_TRACKER__WORK — unsupervised support added, needs real-world testing
- jcodemunch-mcp — next session: connect MCP, explore tools, design skill
