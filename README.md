# Skills

## Nullable skills

Skills for writing code that loosely follows James Shore's [nullable pattern](https://www.jamesshore.com/v2/projects/nullables/testing-without-mocks).

To add to a project:

```sh
bunx skills add danielbush/skills --agent claude-code --skill nullables-refactor
bunx skills add danielbush/skills --agent claude-code --skill nullables-test
bunx skills add danielbush/skills --agent claude-code --skill work-tracker
```

### `nullables-refactor`

Analyzes a file and produces a refactoring plan. Classifies code by side-effect boundary (PURE, IN_MEMORY, OUTSIDE_WORLD), identifies HARDWIRED_INFRA, determines whether code should be an INFRASTRUCTURE_WRAPPER or a NULLABLE_CLASS, checks CREATE_BOUNDARY_RULE compliance, and decides on DELAYED_INSTANTIATION.

### `nullables-test`

Writes illustrative tests after refactoring. Checks preconditions (all HARDWIRED_INFRA replaced by INJECTED_INFRA, every dependency has `.createNull()`, recursive nullability down to the leaves), then writes narrow, sociable, state-based tests using `.createNull()`. Tests illustrate the system's concepts and architecture, not just coverage.

See `docs/vocab/coding.md` for the shared vocabulary these skills use.

#### Eval suite

The refactoring skill has 4 test fixtures in `skills/nullables-refactor/evals/`:

| Fixture | Tests |
|---------|-------|
| `hardwired-function.ts` | Standalone functions calling `fetch` — should be flagged as HARDWIRED_INFRA and converted to an INFRASTRUCTURE_WRAPPER |
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
  .sessions/       → per-user session logs, enables "let's continue"
  work/
    BACKLOG.md     → prioritised backlog (grep-friendly)
    discussion/    → big picture, themes, motivations
    active/        → current work items
```
