# AGENTS.md

This repo explores the human-agent interface for managing code and technical systems. It contains skills, specs, and design artifacts.

## Continuing a session

When the human says "let's continue" or similar:

1. Read the last entry in [SESSIONS.md](SESSIONS.md)
2. Read [TODO.md](TODO.md)
3. Summarize where we left off and what's next
4. Ask which thread to pick up (if multiple are active)

## How to orient

- [PRD.md](PRD.md) — where we're going (goals, north star)
- [TODO.md](TODO.md) — what's in flight right now
- [SESSIONS.md](SESSIONS.md) — what happened recently
- [specs/](specs/) — why things were done (decisions, reasoning, discussion)

## Memory model

These artifacts serve different time horizons:

- **Current context** (conversation) — short-term, resets each session
- **SESSIONS.md** — short-to-mid-term, bridges sessions, enables continuity
- **specs/** — mid-to-long-term, preserves decisions and reasoning
- **Vocabulary, architecture narratives, principles** — long-term, the durable shared understanding

## Vocabulary

Use vocabulary terms (UPPER_SNAKE_CASE) from `docs/vocabulary.md` when they exist. Always use them in agent output — the human prefers the shared language to be "on" so both sides stay in the same conceptual frame.

## Skills

- `skills/nullables-refactor/` — analyze a file and plan nullable refactoring (side-effect boundary classification, HARDWIRED_INFRA detection, CREATE_BOUNDARY_RULE checks)
- `skills/nullables-test/` — write illustrative tests after refactoring (precondition checks, recursive nullability verification, state-based tests via .createNull)
- `skills/effect-ts/` — effect-ts reference
- `sandbox/nullables-skill/` — original monolithic nullables skill (reference only, being superseded)

<!-- opensrc:start -->

## Source Code Reference

Dependencies source code is in `opensrc/` (see `opensrc/sources.json` for available packages). Fetch more with `npx opensrc <package>` (also supports `pypi:`, `crates:`, or `owner/repo`).

<!-- opensrc:end -->
