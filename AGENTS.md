# AGENTS.md

This repo explores the human-agent interface for managing code and technical systems. It contains skills, work artifacts, and design documents.

## Central themes

Strategic and thematic thinking lives in `work/discussion/`. This is where human and agent sharpen their understanding about what they're doing — not tickets or deliverables, but the ongoing process that produces vocabulary, architecture, and principles over time.

The core themes driving this repo are described fully in [`work/discussion/discussion-driven-development.md`](work/discussion/discussion-driven-development.md):

1. **Shared understanding** — vocabulary, architecture narratives, and orientation that enable compressed human-agent communication
2. **Design patterns** — encoded patterns (like nullables) the agent can unpack from short instructions into concrete changes
3. **Institutional memory** — preserving the *why* behind decisions so neither side contradicts past reasoning
4. **Project collaboration framework** — standard artifacts (AGENTS.md, PRD.md, SESSIONS.md, work/) that bridge sessions and give structure

These themes connect through a meta-idea: humans and agents both work well at high compression, but only when there's shared vocabulary, encoded patterns, and memory of why.

See also [`work/discussion/20260320.chore.knowledge-layers.md`](work/discussion/20260320.chore.knowledge-layers.md) for how different kinds of knowledge (world-building, skills, strategic thinking) relate to each other.

## Starting a session

When the human starts a new conversation (greeting, "hello", "let's continue", or similar), offer these options:

1. **Continue where we left off?** — read the last entry in [SESSIONS.md](SESSIONS.md) and scan `work/active/` and `work/discussion/` to summarise current state and ask which thread to pick up
2. **Review the backlog?** — scan [`work/BACKLOG.md`](work/BACKLOG.md) (`grep -A 3 "^## "` for quick index)
3. **Go over the major themes and higher-order motivations?** — read [`work/discussion/discussion-driven-development.md`](work/discussion/discussion-driven-development.md) and walk through it

When continuing, always read `work/active/` and `work/discussion/` to orient.

## How to orient

- [SESSIONS.md](SESSIONS.md) — what happened recently
- [work/discussion/](work/discussion/) — strategic thinking, thematic sharpening, evolving understanding
- [work/active/](work/active/) — work in progress, being delivered
- [work/BACKLOG.md](work/BACKLOG.md) — quick sketches of future work, prioritised (`grep -A 3 "^## "` for quick index)

## Memory model

These artifacts serve different time horizons:

- **Current context** (conversation) — short-term, resets each session
- **SESSIONS.md** — short-to-mid-term, bridges sessions, enables continuity
- **work/** — mid-to-long-term, preserves decisions and reasoning
- **Vocabulary, architecture narratives, principles** — long-term, the durable shared understanding

## Vocabulary

Use vocabulary terms (UPPER_SNAKE_CASE) from `docs/vocabulary.md` when they exist. Always use them in agent output — the human prefers the shared language to be "on" so both sides stay in the same conceptual frame.

## Skills

- `skills/nullables-refactor/` — analyze a file and plan nullable refactoring (side-effect boundary classification, HARDWIRED_INFRA detection, CREATE_BOUNDARY_RULE checks)
- `skills/nullables-test/` — write illustrative tests after refactoring (precondition checks, recursive nullability verification, state-based tests via .createNull)
- `skills/effect-ts/` — effect-ts reference
- `skills/work-tracker/` — create and manage work items in `work/`, bootstraps structure on first use
- `sandbox/nullables-skill/` — original monolithic nullables skill (reference only, being superseded)

<!-- opensrc:start -->

## Source Code Reference

Dependencies source code is in `opensrc/` (see `opensrc/sources.json` for available packages). Fetch more with `npx opensrc <package>` (also supports `pypi:`, `crates:`, or `owner/repo`).

<!-- opensrc:end -->
