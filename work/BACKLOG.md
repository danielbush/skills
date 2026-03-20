# Backlog

Items near the top get worked on first. Each item is an initial draft — improve wording and break out into individual spec files when ready to work on.

Grep-friendly: `grep -A 3 "^## " work/BACKLOG.md` gives a quick index.

---

## feat: Architecture narrative skill
- id: ARCHITECTURE_NARRATIVE
- drafted: 2026-03-20
- summary: Port the story-telling approach to explaining architecture — how a system is structured, told as a narrative a human can follow

The jsed project has a working example of this: the AGENTS.md (`/Users/danb/projects/@oneput/packages/jsed/AGENTS.md`) contains the process/approach — how the narrative is built up — and `docs/architecture.md` (`/Users/danb/projects/@oneput/packages/jsed/docs/architecture.md`) is the result — a bottom-up story where each section depends only on what came before.

The skill (or convention) should help an agent produce and maintain an architecture narrative for any codebase. Key aspects to port: the story-telling structure, the bottom-up dependency ordering, the connection to vocabulary terms, and how it serves as orientation for humans returning to the codebase.

Relates to: SHARED_UNDERSTANDING, ENTRY_POINT_QUESTIONS

## feat: Entry-point questions
- id: ENTRY_POINT_QUESTIONS
- drafted: 2026-03-20
- summary: FAQ-style questions as orientation handles on top of vocabulary + architecture narrative

Curated questions that give humans "doors into" a system — more discoverable than a term list. Examples from jsed: "how does CURSOR traverse TOKENs?", "explain the difference between FOCUS and CURSOR", "how does tokenization work — shallow or deep?"

Agent can answer these when asked, or offer them proactively when the human is reorienting (returning to a codebase, unfamiliar area, been away). Explore how these get captured, maintained, and surfaced — likely as a section in the package AGENTS.md or a dedicated file.

## feat: Spec scanner skill
- id: SPEC_SCANNER
- drafted: 2026-03-20
- summary: Quick orientation by scanning frontmatter of spec files sorted by date

Scan the first few lines of spec files sorted by date for a fast "what's been happening" view without reading full specs. Lightweight alternative to reading SESSIONS.md.

## feat: mcporter skill
- id: MCPORTER
- drafted: 2026-03-20
- summary: Interact with MCP servers without loading the full MCP into context

Uses `npx -y mcporter list <server>` to discover tools and `npx -y mcporter call <server>.<tool>(<args>)` to invoke them. Example: `mcporter call deepwiki.ask_question(repoName: "owner/repo", question: "...")`.

Reference: https://youtu.be/wVe3XOnio7M

## feat: opensrc skill
- id: OPENSRC
- drafted: 2026-03-20
- summary: Encourage agent to download important dependencies and read them proactively

Agent should proactively use `npx opensrc <package>` to download and read important dependencies/references unless an up-to-date skill already covers it. Make this a natural part of the agent's workflow rather than something the human has to prompt.

## chore: Explore pi harness
- id: PI_HARNESS
- drafted: 2026-03-20
- summary: Investigate agentic workflow framework for coding and devops flows

Framework from https://github.com/badlogic/pi-mono (also used in openclaw). Interested in both coding workflows and non-coding flows for managing whole systems (devops).

## chore: Vocabulary pass (broader)
- id: VOCAB_BROADER
- drafted: 2026-03-18
- summary: Extend docs/vocabulary.md beyond nullables — deep modules, on-demand vs ambient skills, the four themes

## chore: Generalize @oneput skills
- id: GENERALIZE_ONEPUT
- drafted: 2026-03-17
- summary: Extract remember, summarize, symbol-lookup from @oneput for potential reuse

Look at `/Users/danb/projects/@oneput/.agents/skills/remember`, `summarize`, `symbol-lookup` for extraction into this repo.

## chore: Convert jsed to generalized skill
- id: GENERALIZE_JSED
- drafted: 2026-03-17
- summary: Extract jsed orientation skill into a generalized skill in this repo

Relates to SHARED_UNDERSTANDING. `/Users/danb/projects/@oneput/.agents/skills/jsed` → new generalized skill here.

## chore: Architecture skills PRD
- id: ARCH_SKILLS_PRD
- drafted: 2026-03-07
- summary: Original exploration of architectural principles as always-on (CLAUDE.md) vs actionable workflows (skills)

Identifies the two-layer approach: Layer 1 is CLAUDE.md principles (A-Frame, deep modules, consumer-defined interfaces, value objects, error mapping), Layer 2 is on-demand skills (nullables). Previously at `work/discussion/architecture-skills-prd.md`. Relates to DESIGN_PATTERNS.
