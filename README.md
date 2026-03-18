# Skills

## Nullable skill

The main skill of interest here is the nullables skill.

This is a skill for writing code that loosely follows James Shore's [nullable pattern](https://www.jamesshore.com/v2/projects/nullables/testing-without-mocks)

To add to a project:

```sh
bunx skills add danielbush/skills --agent claude-code --skill nullables
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
