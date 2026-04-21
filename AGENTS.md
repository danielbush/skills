# AGENTS.md

This repo explores the human-agent interface for managing code and technical systems. It contains skills, work artifacts, and design documents.

## Purpose

This repo is about overlapping memory and context management to create agents that know what you were doing recently and last time, what was on your mind, what was bothering you, and what the project is about and how it works.

The themes and thinking behind this live in `docs/`:

- [`docs/discussion-driven-development.md`](docs/discussion-driven-development.md) — the four central themes (shared understanding, design patterns, institutional memory, project collaboration) and the higher-order motivations behind them
- [`docs/themes.md`](docs/themes.md) — how different layers of knowledge (world-building, skills, strategic thinking) relate to each other
- [`docs/session-continuity.md`](docs/session-continuity.md) — the vision for "let's continue" — how work/.sessions/, work/, and docs/ combine to orient the agent at session start

<!-- BEGIN work-tracker:session-bookends -->
## Session bookends

This project uses the work-tracker skill for session continuity. Sessions have two bookends — a **resume** at the start and a **wrap-up** at the end — and the agent should proactively help with both.

### At the start of a session

If `work/active/` has items and the human opens with a generic greeting ("hi", "hello", "ok let's go", or jumps straight into a new topic), **proactively summarise what happened last time before anything else**:

1. Read the latest entry in `work/.sessions/$USER.md` — the "What happened" and "On my mind" sections.
2. Scan `work/active/` — Status + Tasks from each active item.
3. Tell the human: "Last time we …; what was on your mind was …; active threads are …" — concrete, not a question.
4. Then offer: "want to pick one up, review the backlog, or start something new?"

The goal is to *orient* the human, not wait for them to ask. If they've said "where were we" explicitly, you're already doing this — just do it unprompted too when the signals above are present.

### At the end of a session

Remind the human early-ish that the session has a wrap-up phase: when they wind down, they should say "let's wrap up" / "let's stop here" / "summarise" so the agent can take notes. Without this signal, the session ends and nothing gets captured.

When the human signals winding down ("ok that's enough", "I'm done", "going to stop"), prompt them:

> "Want me to wrap up the session? I'll update active tickets and append to the session log."

If they say yes, run the work-tracker summarise flow (update work items + append to `work/.sessions/$USER.md` with What happened / On my mind / Open threads).

Don't force either bookend. If the human declines or redirects, drop it — but don't silently skip the wrap-up; a brief "want me to capture this?" at the end is cheap and easy to decline.
<!-- END work-tracker:session-bookends -->

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
