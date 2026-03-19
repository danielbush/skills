# PRD — Human-Agent Interface for Technical Systems

## North star

Explore and build tooling for the divide between human and agent in code and system design — where the human steers and the agent implements, but both need to stay aligned.

If you need more detail, see [discussion-driven-development.md](specs/discussion/discussion-driven-development.md) .

## Core ideas

### Compression and grounding

Humans and agents both work well at high compression — a few words can trigger complex, precise action. But compression only works when grounded. Two mechanisms:

- **Vocabulary** — named concepts that compress a system into something a human can carry and an agent can act on precisely
- **Key examples** — concrete instances that ground abstract vocabulary in reality; without them, terms are just labels

The agent compresses (distills systems into vocabulary, narratives, principles). The human grounds (validates understanding through examples, tests, visual models). Both directions are necessary.

### The human as technical director

As agents handle more implementation, the human's role shifts to steering. The risk is drift — losing enough technical understanding to make good decisions. The response is not "read all the code" but "stay grounded in the right concepts." Vocabulary, architecture narratives, and key examples keep the human technically engaged without requiring them to implement.

### Beyond single codebases

These ideas are not scoped to one repo. A human managing multiple systems needs the same compression and grounding across all of them. The current implementation (markdown files) is prototyping the structures — vocabulary, principles, sessions, decisions — that could eventually live in a graph-backed system where relationships across codebases are queryable.

## Current work streams

See `specs/active/` for current work and [`specs/BACKLOG.md`](specs/BACKLOG.md) for the prioritised queue. Four active threads:

1. **Shared understanding** — generalize the vocab + architecture narrative pattern
2. **Design patterns** — separate principles (CLAUDE.md) from workflows (skills)
3. **Specs system** — generalize /summarize and /remember across projects
4. **Collaboration framework** — standard project files (AGENTS.md, PRD.md, SESSIONS.md, specs/) for session continuity
