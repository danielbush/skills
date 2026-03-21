# Backlog

Items near the top get worked on first. Each item is an initial draft — improve wording and break out into individual spec files when ready to work on.

Grep-friendly: `grep -A 3 "^## " work/BACKLOG.md` gives a quick index.

---

## feat: Research tool skill
- id: RESEARCH_TOOL__WORK
- drafted: 2026-03-22
- summary: Skill for researching topics using web search, papers, and content extraction — with small subagents (haiku, gpt-5.4-mini) for processing

Build a research skill that can go through content and identify relevant things. Key inputs and references:

- **Maximilian's video**: on Pi Agentic harness https://youtu.be/wVe3XOnio7M?si=X18pLWiFdyYQlK9- — I have a screenshot somewhere
- **Small subagents**: use mini/small models (haiku, gpt-5.4-mini) to process and filter content cheaply
- **Paper research**: marimo's implement-paper skill — https://github.com/marimo-team/skills/blob/main/skills/implement-paper/SKILL.md
- **Search approaches**:
  - Gemini may have built-in search — https://ai.google.dev/gemini-api/docs/google-search
  - Check openclaw skills for how it does search
- **Other tools**: firecrawl, perplexity — https://cline.bot/blog/5-tool-mcp-starter-pack-for-cline

Relates to: PI_HARNESS__WORK, CODE_RESEARCH__WORK

## feat: Code research tool skill
- id: CODE_RESEARCH__WORK
- drafted: 2026-03-22
- summary: Skill for researching codebases and libraries — mcporter + deepwiki, opensrc + AI docs, small subagents for processing

Tools for understanding unfamiliar code. Key approaches:

- **Small subagents**: use mini/small models (haiku, gpt-5.4-mini) to go through content and identify things
- **mcporter + deepwiki**: use mcporter to query deepwiki for repo-level understanding (relates to MCPORTER__WORK)
- **opensrc**: proactively download and read important dependencies via `npx opensrc <package>` (supports `pypi:`, `crates:`, `owner/repo`). Use a subagent to find and surface relevant docs. Should be a natural part of agent workflow, not something the human has to prompt.
- Relates to: RESEARCH_TOOL__WORK, MCPORTER__WORK

## feat: YouTube transcript skill
- id: YOUTUBE_TRANSCRIPT__WORK
- drafted: 2026-03-22
- summary: Skill to pull and process YouTube transcripts — check what openclaw does

Extract transcripts from YouTube videos for processing. Investigate what openclaw's approach is. Useful for the research tool (RESEARCH_TOOL__WORK) when references are video content.

## chore: Rethink docs/ structure
- id: DOCS_STRUCTURE__WORK
- drafted: 2026-03-21
- summary: docs/ mixes different concerns — vocabulary.md is about refactoring/testing code, other docs are about bigger themes; needs reworking over time

`docs/vocabulary.md` is really a vocab for the nullables pattern (refactoring and testing code). The other docs (`discussion-driven-development.md`, `themes.md`) are about the bigger themes driving this repo. These are different kinds of thing living in the same place. Not urgent, but worth thinking about how to organise as more docs accumulate.

## feat: Architecture narrative skill
- id: ARCHITECTURE_NARRATIVE__WORK
- drafted: 2026-03-20
- summary: Port the story-telling approach to explaining architecture — how a system is structured, told as a narrative a human can follow

The jsed project has a working example of this: the AGENTS.md (`/Users/danb/projects/@oneput/packages/jsed/AGENTS.md`) contains the process/approach — how the narrative is built up — and `docs/architecture.md` (`/Users/danb/projects/@oneput/packages/jsed/docs/architecture.md`) is the result — a bottom-up story where each section depends only on what came before.

The skill (or convention) should help an agent produce and maintain an architecture narrative for any codebase. Key aspects to port: the story-telling structure, the bottom-up dependency ordering, the connection to vocabulary terms, and how it serves as orientation for humans returning to the codebase.

Relates to: SHARED_UNDERSTANDING__WORK, ENTRY_POINT_QUESTIONS__WORK

## feat: Entry-point questions
- id: ENTRY_POINT_QUESTIONS__WORK
- drafted: 2026-03-20
- summary: FAQ-style questions as orientation handles on top of vocabulary + architecture narrative

Curated questions that give humans "doors into" a system — more discoverable than a term list. Examples from jsed: "how does CURSOR traverse TOKENs?", "explain the difference between FOCUS and CURSOR", "how does tokenization work — shallow or deep?"

Agent can answer these when asked, or offer them proactively when the human is reorienting (returning to a codebase, unfamiliar area, been away). Explore how these get captured, maintained, and surfaced — likely as a section in the package AGENTS.md or a dedicated file.

## feat: Spec scanner skill
- id: SPEC_SCANNER__WORK
- drafted: 2026-03-20
- summary: Quick orientation by scanning frontmatter of spec files sorted by date

Scan the first few lines of spec files sorted by date for a fast "what's been happening" view without reading full specs. Lightweight alternative to reading SESSIONS.md.

## feat: mcporter skill
- id: MCPORTER__WORK
- drafted: 2026-03-20
- summary: Interact with MCP servers without loading the full MCP into context

Uses `npx -y mcporter list <server>` to discover tools and `npx -y mcporter call <server>.<tool>(<args>)` to invoke them. Example: `mcporter call deepwiki.ask_question(repoName: "owner/repo", question: "...")`.

Reference: https://youtu.be/wVe3XOnio7M

## chore: Explore pi harness
- id: PI_HARNESS__WORK
- drafted: 2026-03-20
- summary: Investigate agentic workflow framework for coding and devops flows

Framework from https://github.com/badlogic/pi-mono (also used in openclaw). Interested in both coding workflows and non-coding flows for managing whole systems (devops).

## chore: Explore jdocmunch-mcp
- id: JDOCMUNCH__WORK
- drafted: 2026-03-20
- summary: Understand what jdocmunch-mcp does and whether it can index md files (work/, docs/) and vocab terms

Investigate jdocmunch-mcp — what it does generally, and specifically: can it index all markdown files including `work/` and `docs/`? Can it index on vocabulary terms (UPPER_SNAKE_CASE) at all? Could be useful for searching across work items, themes, and vocabulary. Also explore using mcporter (MCPORTER__WORK) to interact with it without loading the full MCP into context.

## chore: Vocabulary pass (broader)
- id: VOCAB_BROADER__WORK
- drafted: 2026-03-18
- summary: Extend docs/vocabulary.md beyond nullables — deep modules, on-demand vs ambient skills, the four themes

## chore: Generalize @oneput skills
- id: GENERALIZE_ONEPUT__WORK
- drafted: 2026-03-17
- summary: Extract remember, summarize, symbol-lookup from @oneput for potential reuse

Look at `/Users/danb/projects/@oneput/.agents/skills/remember`, `summarize`, `symbol-lookup` for extraction into this repo.

## chore: Convert jsed to generalized skill
- id: GENERALIZE_JSED__WORK
- drafted: 2026-03-17
- summary: Extract jsed orientation skill into a generalized skill in this repo

Relates to SHARED_UNDERSTANDING__WORK. `/Users/danb/projects/@oneput/.agents/skills/jsed` → new generalized skill here.

## chore: Architecture skills PRD
- id: ARCH_SKILLS_PRD__WORK
- drafted: 2026-03-07
- summary: Original exploration of architectural principles as always-on (CLAUDE.md) vs actionable workflows (skills)

Identifies the two-layer approach: Layer 1 is CLAUDE.md principles (A-Frame, deep modules, consumer-defined interfaces, value objects, error mapping), Layer 2 is on-demand skills (nullables). Previously at `work/discussion/architecture-skills-prd.md`. Relates to DESIGN_PATTERNS__WORK.
