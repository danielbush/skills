# AGENTS.md

This repo explores the human-agent interface for managing code and technical systems. It contains skills, work artifacts, and design documents.

## Purpose

This repo is about overlapping memory and context management to create agents that know what you were doing recently and last time, what was on your mind, what was bothering you, and what the project is about and how it works.

The themes and thinking behind this live in `docs/`:

- [`docs/discussion-driven-development.md`](docs/discussion-driven-development.md) — the four central themes (shared understanding, design patterns, institutional memory, project collaboration) and the higher-order motivations behind them
- [`docs/themes.md`](docs/themes.md) — how different layers of knowledge (world-building, skills, strategic thinking) relate to each other
- [`docs/session-continuity.md`](docs/session-continuity.md) — the vision for "let's continue" — how work/.sessions/, work/, and docs/ combine to orient the agent at session start

## Vocabulary

Vocabulary files live in `docs/vocab/`. Use vocabulary terms (UPPER_SNAKE_CASE) from the relevant vocab file when they exist. Always use them in agent output — the human prefers the shared language to be "on" so both sides stay in the same conceptual frame.

## Skills

Skills are portable — they carry knowledge about *how to do things* across projects. Install with `bunx skills add danielbush/skills --agent claude-code --skill <name>`. They activate based on what the human says (semantic matching against the description).

### work-tracker — session continuity, work items, and memory

**Theme: project collaboration + institutional memory**

Handles session start/continue ("let's continue", "where were we"), creating and managing work items ("create a ticket", "let's track this"), summarising sessions, and searching past work ("remember when we..."). Bootstraps `work/` on first use.

This is the skill that creates the overlapping memory layers: `work/.sessions/` for short-term (what happened, what was on your mind), `work/` for mid-term (active tickets, backlog), sitting on top of `docs/` for long-term (vocab, architecture, themes).

### nullable-architecture — refactor and test in the Nullables style

**Theme: design patterns**

Guidance for refactoring code and writing tests in the Nullables style: choosing between `new`, `.create()`, and `.createNull()`, introducing infrastructure wrappers at the environment boundary, adding behavior simulation and output tracking, and writing narrow, sociable, example-driven, state-based tests without mocks or spies.

### logic-sandwich — refactor orchestrators into gather / decide / apply

**Theme: design patterns**

Use when refactoring a top-level orchestrator or mediator method so it gathers state at the edges, computes a pure intent in the middle, and applies effects at the bottom. Especially useful in deep-modules codebases where top-level modules should stay readable and orchestration-focused.

### jcodemunch — symbol-aware code search and exploration

**Theme: shared understanding**

Query jcodemunch-mcp indexed codebases via `bunx mcporter` for symbol-aware code intelligence (definitions, references, call chains, dependency/impact analysis) beyond what plain grep offers. Prefer this over raw file reads when exploring code structure, relationships, or usage patterns.

### opensrc — fetch and reference dependency source code

Fetch source for npm, PyPI, crates.io, or GitHub repos into `opensrc/`. Use when you need to understand how a package works internally.

## Sandbox

`sandbox/` holds in-progress or superseded skills and reference material. Not installed as active skills.

- `sandbox/nullables/` — reference notes and diagrams on James Shore's Nullables pattern (nullables.md, testing-without-mocks-diagrams.md)
- `sandbox/nullables-skill/` — original monolithic nullables skill, superseded by `skills/nullable-architecture`
- `sandbox/nullables-refactor/` — analyse a file and produce a Nullables refactoring plan (classifies PURE / IN_MEMORY / OUTSIDE_WORLD, identifies HARDWIRED_INFRA, checks CREATE_BOUNDARY_RULE and DELAYED_INSTANTIATION). Includes evals and references.
- `sandbox/nullables-test/` — write illustrative, narrow, sociable, state-based tests after a Nullables refactor
- `sandbox/nullables-refactor-workspace/` — iteration workspace with benchmarks and eval cases (already-compliant, hardwired-function, mixed-class, partial-nullable) used to develop the refactor skill
