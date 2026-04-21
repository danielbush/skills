## 2026-04-01

### What happened
- Trimmed work-tracker skill description from 1053 to 525 chars — a model refused to load it because it exceeded the 1024-char limit from the Agent Skills spec
- Fetched `agentskills/agentskills` via opensrc — confirmed the spec defines hard limits (name: 64, description: 1024, compatibility: 500) with a Python validator
- Noted the skill-creator doesn't know about the 1024-char limit — neither when drafting descriptions nor during the description optimization loop. Didn't address yet.
- Moved opensrc instructions from CLAUDE.md into a new `skills/opensrc/` skill
- Spotted `skills/nullables-refactor-workspace/` (stale eval output from March 18) sitting in skills/ — not cleaned up yet

### On my mind
- **Skill-creator needs the 1024-char constraint** — both in the Skill Writing Guide (so it knows when drafting) and in the description optimization loop (so it rejects oversized candidates). Didn't get to it this session.
- **Work-tracker MCP server** — idea to back the ticket system with a db via an MCP server, skill talks to it via mcporter. Added to backlog as WORK_TRACKER_MCP__WORK. Open questions: sqlite vs something else, does the file format remain source of truth or does the db?
- **nullables-refactor-workspace** — stale eval sitting in skills/, should be moved or gitignored
- The other open threads from last time (NULLABLES_SKILLS__WORK AAA pattern, JCODEMUNCH__WORK eval, unsupervised tickets) weren't touched

### Open threads
- WORK_TRACKER__WORK — description trimmed; MCP server idea in backlog
- WORK_TRACKER_MCP__WORK — new backlog item, unexplored
- NULLABLES_SKILLS__WORK — unchanged, AAA pattern still outstanding
- JCODEMUNCH__WORK — unchanged
- skill-creator 1024-char constraint — not yet addressed

## 2026-04-21

### What happened
- Refined "What To Test" in `skills/nullable-architecture/SKILL.md`. Scoped explicitly to **unit tests**. Replaced the "intermediate code" framing with an interaction-criterion: did the test orchestrate several subcomponents to produce one outward behaviour? If so, it's encoding internals. Pushed tests toward a barbell (top-level sociable via nullables+DI, or low-level narrow single-responsibility). Added the "mirroring same behaviour at multiple levels" smell.
- Added session-bookends behaviour to work-tracker. Skill now ships `agents-snippet.md` with a delimited block (`<!-- BEGIN/END work-tracker:session-bookends -->`) that projects paste into their AGENTS.md / CLAUDE.md. Tells the agent to *proactively* summarise last session at start (not wait for "where were we") and to remind the human to signal wrap-up so notes get captured. Pasted the block into this repo's own CLAUDE.md.
- Created NULLABLES_WHAT_TO_TEST__WORK to track the nullables refinement.
- Updated auto-memory `feedback_testing_level.md` to match the refined test-level framing.

### On my mind
- **Tracking work in the repo matters** — memory is fine, but the primary concern is what's in `work/`. That drove the conversation about session bookends: the agent should prompt, not wait.
- **Skills can't ship hooks** — hooks are a harness feature (Claude Code settings.json), not part of the Agent Skills spec. So behavioural nudges have to travel via AGENTS.md snippets. The delimited-block pattern is the solution — managed content with markers so it can be updated in place.
- **Proactive agent behaviour is the lever** — "where were we" as a user phrase is fine, but relying on the human to remember it is fragile. AGENTS.md is always loaded, so instructions there shape behaviour every turn. Worth thinking about what other collaboration patterns should be delivered this way.
- The three prior open threads (skill-creator 1024-char, WORK_TRACKER_MCP, JCODEMUNCH direction) still haven't been touched.

### Open threads
- WORK_TRACKER__WORK — session-bookends behaviour added; snippet + CLAUDE.md integration done
- NULLABLES_WHAT_TO_TEST__WORK — new, drafted the refined section; may need to ripple into `sandbox/nullables-test/`
- WORK_TRACKER_MCP__WORK — still unexplored
- JCODEMUNCH__WORK — still unchanged; three directions to pick between
- skill-creator 1024-char constraint — still not addressed
- OVERVIEW_SKILL__WORK — untouched this session
