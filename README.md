# Skills

## Nullable skills

Skills for writing code that loosely follows James Shore's [nullable pattern](https://www.jamesshore.com/v2/projects/nullables/testing-without-mocks).

To add to a project:

```sh
bunx skills add danielbush/skills --agent claude-code --skill nullables
bunx skills add danielbush/skills --agent claude-code --skill nullables-refactor
```

### `nullables`

The main implementation skill. Guides writing and testing code using the nullable pattern: DUAL_FACTORY (`.create`/`.createNull`), INFRASTRUCTURE_WRAPPERs, EMBEDDED_STUBs, and state-based testing without mocks.

### `nullables-refactor`

Analyzes a file and produces a refactoring plan. Classifies code by side-effect boundary (PURE, IN_MEMORY, OUTSIDE_WORLD), identifies HARDWIRED_INFRASTRUCTURE, determines whether code should be an INFRASTRUCTURE_WRAPPER or a NULLABLE_CLASS, checks CREATE_BOUNDARY_RULE compliance, and decides on DELAYED_INSTANTIATION.

See `docs/vocabulary.md` for the shared vocabulary these skills use.

#### Eval suite

The refactoring skill has 4 test fixtures in `skills/nullables-refactor/evals/`:

| Fixture | Tests |
|---------|-------|
| `hardwired-function.ts` | Standalone functions calling `fetch` — should be flagged as HARDWIRED_INFRASTRUCTURE and converted to an INFRASTRUCTURE_WRAPPER |
| `mixed-class.ts` | Class mixing PURE logic with `fs`/`fetch` calls — should extract wrappers and become a NULLABLE_CLASS |
| `partial-nullable.ts` | Class with DUAL_FACTORY but CREATE_BOUNDARY_RULE violations — `.create()` called inside instance methods |
| `already-compliant.ts` | Properly structured INFRASTRUCTURE_WRAPPER + NULLABLE_CLASS — should report compliance, no false positives |

Run evals with:

```sh
task evals:nullables-refactor
```

## Development

A place to incubate and play with skills or related things to do with context.  Very much in alpha.

Say "Let's continue" to pick up from last time - this is controlled in AGENTS.md.

AGENTS.md is the "always-on" layer, skills are the "on-demand" layer. Same idea as the principles-vs-workflows split in Theme 2.

```
  CLAUDE.md        → points to AGENTS.md (no longer a symlink)
  AGENTS.md        → how to work here, memory model, vocab preference
  PRD.md           → north star: human-agent interface, compression + grounding
  TODO.md          → four active work streams with links to specs
  SESSIONS.md      → this session logged as the first entry
  specs/
    discussion/    → big picture + old PRD
    active/        → four actionable specs (including the framework itself)
```
