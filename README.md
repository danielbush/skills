# Skills

Primarily focuses on coding skills and prmopt/context engineering ideas.
A place to incubate and play with skills or related things to do with context.  Very much in alpha.

## Coding skills

### Design philosophy

These ideas are heavily inspired by others' work — in particular James Shore's [Testing Without Mocks](https://www.jamesshore.com/v2/projects/nullables/testing-without-mocks) (nullables, infrastructure wrappers, embedded stubs, logic sandwich) and John Ousterhout's *A Philosophy of Software Design* (deep modules). What's here is our interpretation, not a faithful reproduction — we've adapted the ideas to fit how we build. Go read the originals if you want the real thing.

Here are some of the ideas...

- **Deep modules** — top-level code stays simple and orchestrates; detail lives lower down
- **Logic sandwich** — orchestrators gather state, compute a pure intent, apply effects — no mixing
- **Infrastructure wrappers** — the environment boundary is wrapped in production code with an "off switch" (embedded stubs), not mocked in tests
- **Nullable components** - classes that use infrastructure ultimately resulting in hitting an infrastructure wrapper are "nullable"
- **Dependency injection** - injecting infrastructure dependencies; using nulled or nullable dependencies in units tests, allowing almost every line of code to be exercised at unit-test time
- **state-based unit tests** - as opposed to behavioural tests; to avoid tests locking in your design choices making refactors prohibitive
- **sociable unit tests** - especially for system behaviours and usually at a high level within the code; also reduces lock in when refactoring;

The payoff: every line of production code runs in tests, fast and deterministic, with no mocks, no patching, no test-only seams.

The anti-pattern that breaks all of this is **embedded infrastructure** — importing something at the top of a file and calling it inline within a function or method. That one habit is what creates the need for mocking frameworks; or, you give up on unit testing these and resort to an over-reliance on live / e2e testing resulting in slow long running tests with flakey hard-to-understand errors.

### Vocabulary and architecture (speculative)

The design philosophy above works better when human and agent share a named language for it. We're exploring the idea of deliberately engineering vocabulary (`docs/vocab/`) — UPPER_SNAKE_CASE terms like INFRASTRUCTURE_WRAPPER, EMBEDDED_STUB, DEEP_MODULE — so both sides can communicate at high compression without losing precision. Paired with a bottom-up architecture narrative (`docs/architecture.md`) where each section depends only on what came before, these become "world-building" artifacts: ambient project knowledge that lets an agent orient itself without being told everything from scratch each session. This isn't a skill yet — it's a direction. See `work/discussion/shared-understanding-skill.md` and `docs/themes.md` for the longer conversation.

### Skill: Nullable architecture

Testing style inspired by James Shore's [nullable pattern](https://www.jamesshore.com/v2/projects/nullables/testing-without-mocks). Covers infrastructure wrappers, embedded stubs, configurable responses, output tracking, behavior simulation, and sociable state-based tests without mocks. Also covers the embedded infrastructure anti-pattern and why injection is preferred over mocking frameworks.

```sh
bunx skills add danielbush/skills --agent claude-code --skill nullable-architecture
```

### Skill: Logic sandwich

Refactor top-level orchestrator methods into three layers: gather state at the edges, compute a pure intent in the middle, apply effects at the bottom. Fits deep-modules codebases where top-level files should stay simple and high-level.

```sh
bunx skills add danielbush/skills --agent claude-code --skill logic-sandwich
```

### Skill: jcodemunch (WIP)

Code search and exploration using [jcodemunch-mcp](https://github.com/jgravelle/jcodemunch-mcp) via [mcporter](https://github.com/steipete/mcporter). Symbol-aware search, dependency graphs, blast radius analysis, and repo orientation — without loading the full MCP server into context.

**Status**: trigger eval showed 0% auto-trigger rate. Claude doesn't consult skills for code exploration tasks because it thinks it can handle them with built-in tools (Read/Grep/Glob). Investigating alternatives — CLAUDE.md instructions, explicit `/jcodemunch` slash command, or hybrid. See JCODEMUNCH__WORK for details.

```sh
bunx skills add danielbush/skills --agent claude-code --skill jcodemunch
```

Requires `jcodemunch-mcp` to be installed (`uvx jcodemunch-mcp`) and repos to be indexed before querying.

## Project management skills

### Skill: Work tracker

Session continuity, work items, and memory. Handles "let's continue", ticket creation, session summarisation, and searching past work.

```sh
bunx skills add danielbush/skills --agent claude-code --skill work-tracker
```

## Development

Uses work-tracker skill.

Say "Let's continue" to pick up from last time - this is controlled in AGENTS.md.

AGENTS.md is the "always-on" layer, skills are the "on-demand" layer. Same idea as the principles-vs-workflows split in Theme 2.

```
  CLAUDE.md        → points to AGENTS.md (no longer a symlink)
  AGENTS.md        → how to work here, memory model, vocab preference
  .sessions/       → per-user session logs, enables "let's continue"
  work/
    BACKLOG.md     → prioritised backlog (grep-friendly)
    discussion/    → things you're not sure about — discuss with an agent or colleague
    active/        → current work items
```
