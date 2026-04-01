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
