# AGENTS.md

This repo explores the human-agent interface for managing code and technical systems. It contains skills, work artifacts, and design documents.

## Purpose

This repo is about overlapping memory and context management to create agents that know what you were doing recently and last time, what was on your mind, what was bothering you, and what the project is about and how it works.

The themes and thinking behind this live in `docs/`:

- [`docs/discussion-driven-development.md`](docs/discussion-driven-development.md) — the four central themes (shared understanding, design patterns, institutional memory, project collaboration) and the higher-order motivations behind them
- [`docs/themes.md`](docs/themes.md) — how different layers of knowledge (world-building, skills, strategic thinking) relate to each other

## Vocabulary

Use vocabulary terms (UPPER_SNAKE_CASE) from `docs/vocabulary.md` when they exist. Always use them in agent output — the human prefers the shared language to be "on" so both sides stay in the same conceptual frame.

## Skills

Skills are portable — they carry knowledge about *how to do things* across projects. Install with `bunx skills add danielbush/skills --agent claude-code --skill <name>`. They activate based on what the human says (semantic matching against the description).

### work-tracker — session continuity, work items, and memory

**Theme: project collaboration + institutional memory**

Handles session start/continue ("let's continue", "where were we"), creating and managing work items ("create a ticket", "let's track this"), summarising sessions, and searching past work ("remember when we..."). Bootstraps `work/` and `.sessions/` on first use.

This is the skill that creates the overlapping memory layers: `.sessions/` for short-term (what happened, what was on your mind), `work/` for mid-term (active tickets, backlog), sitting on top of `docs/` for long-term (vocab, architecture, themes).

### nullables-refactor — analyse and plan nullable refactoring

**Theme: design patterns**

Analyse a file and produce a refactoring plan for the nullables pattern. Classifies code by side-effect boundary (PURE, IN_MEMORY, OUTSIDE_WORLD), identifies HARDWIRED_INFRA, determines INFRASTRUCTURE_WRAPPER vs NULLABLE_CLASS, checks CREATE_BOUNDARY_RULE compliance and DELAYED_INSTANTIATION.

### nullables-test — write illustrative tests after refactoring

**Theme: design patterns**

Write tests for code that follows the nullables pattern. Verifies preconditions (all HARDWIRED_INFRA replaced, every dependency has `.createNull`), then writes narrow, sociable, state-based tests. Tests are illustrations of the system, not just coverage.

### effect-ts — effect-ts reference

Reference material for effect-ts.

### sandbox/nullables-skill/ — original monolithic nullables skill (reference only, being superseded)

<!-- opensrc:start -->

## Source Code Reference

Dependencies source code is in `opensrc/` (see `opensrc/sources.json` for available packages). Fetch more with `npx opensrc <package>` (also supports `pypi:`, `crates:`, or `owner/repo`).

<!-- opensrc:end -->
