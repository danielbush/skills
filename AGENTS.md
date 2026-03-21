# AGENTS.md

This repo explores the human-agent interface for managing code and technical systems. It contains skills, work artifacts, and design documents.

## Central themes

Strategic and thematic thinking lives in `docs/`. This is where human and agent sharpen their understanding about what they're doing — not tickets or deliverables, but the ongoing process that produces vocabulary, architecture, and principles over time.

The core themes driving this repo are described fully in [`docs/discussion-driven-development.md`](docs/discussion-driven-development.md):

1. **Shared understanding** — vocabulary, architecture narratives, and orientation that enable compressed human-agent communication
2. **Design patterns** — encoded patterns (like nullables) the agent can unpack from short instructions into concrete changes
3. **Institutional memory** — preserving the *why* behind decisions so neither side contradicts past reasoning
4. **Project collaboration framework** — standard artifacts (AGENTS.md, .sessions/, work/, docs/) that bridge sessions and give structure

These themes connect through a meta-idea: humans and agents both work well at high compression, but only when there's shared vocabulary, encoded patterns, and memory of why.

Put another way: overlapping memory and context management to create agents that know what you were doing recently and last time, what was on your mind, what was bothering you, and what the project is about and how it works.

See also [`docs/themes.md`](docs/themes.md) for how different kinds of knowledge (world-building, skills, strategic thinking) relate to each other.

## Starting a session

When the human starts a new conversation (greeting, "hello", "let's continue", or similar), offer these options:

1. **Continue where we left off?** — read the last entry in `.sessions/$USER.md` and scan `work/active/` to summarise current state (last session's feel + factual work progress) and ask which thread to pick up
2. **Review the backlog?** — scan [`work/BACKLOG.md`](work/BACKLOG.md) (`grep -A 3 "^## "` for quick index)
3. **Go over the major themes and higher-order motivations?** — read [`docs/discussion-driven-development.md`](docs/discussion-driven-development.md) and walk through it

When continuing, always read `.sessions/$USER.md` (last entry) and `work/active/` to orient.

## How to orient

- [.sessions/](/.sessions/) — per-user session logs (what happened, what was on your mind)
- [docs/](docs/) — vocabulary, themes, strategic thinking, architecture
- [work/active/](work/active/) — work in progress, being delivered
- [work/BACKLOG.md](work/BACKLOG.md) — quick sketches of future work, prioritised (`grep -A 3 "^## "` for quick index)

## Memory model

Overlapping layers of memory create continuity:

- **Current context** (conversation) — short-term, resets each session
- **.sessions/$USER.md** — short-term bridge, what happened last time, what was on your mind
- **work/** — mid-term, tracks active work and progress across sessions
- **docs/** (vocabulary, architecture, themes) — long-term, the durable shared understanding of what the project is and how it works

## Vocabulary

Use vocabulary terms (UPPER_SNAKE_CASE) from `docs/vocabulary.md` when they exist. Always use them in agent output — the human prefers the shared language to be "on" so both sides stay in the same conceptual frame.

## Skills

- `skills/nullables-refactor/` — analyze a file and plan nullable refactoring (side-effect boundary classification, HARDWIRED_INFRA detection, CREATE_BOUNDARY_RULE checks)
- `skills/nullables-test/` — write illustrative tests after refactoring (precondition checks, recursive nullability verification, state-based tests via .createNull)
- `skills/effect-ts/` — effect-ts reference
- `skills/work-tracker/` — work items, session continuity, summarisation, and search across `work/` and `.sessions/`
- `sandbox/nullables-skill/` — original monolithic nullables skill (reference only, being superseded)

<!-- opensrc:start -->

## Source Code Reference

Dependencies source code is in `opensrc/` (see `opensrc/sources.json` for available packages). Fetch more with `npx opensrc <package>` (also supports `pypi:`, `crates:`, or `owner/repo`).

<!-- opensrc:end -->
