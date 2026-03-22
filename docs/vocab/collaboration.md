# Vocabulary — Collaboration

Terms are UPPER_SNAKE_CASE. Use them in agent output to stay in the shared conceptual frame.

## Core Concepts

### DISCUSSION_DRIVEN_DEVELOPMENT
The practice of humans steering and agents implementing, with both sides communicating at a high-compression natural language level. The human relinquishes the decompressed complexity — the conversion to code, math, legalese, formal specifications — that technical humans were traditionally prized for. The agent handles that expansion. The human's value shifts to intent, direction, judgment, and understanding. This only works when there's SHARED_VOCABULARY, encoded patterns, and memory of why. The name is a playful nod to domain-driven design — both share the emphasis on ubiquitous language and naming things well, but DDD was about modelling domains in code; DISCUSSION_DRIVEN_DEVELOPMENT is about the human staying in the conversation while the agent handles the code.

### VOCAB_ENGINEERING
The deliberate practice of identifying and labelling concepts and entities clearly, creating a shared language between human and agent. Not jargon for its own sake — it's a tool for humans who want to maintain precision at the high-compression level where they now operate. Well-engineered vocabulary improves communication in both directions: the human can say more with less, and the agent's output is parseable at a glance. The vocabulary bridges natural language (where humans think) and implementation detail (where agents work).

### CONTINUITY
The general principle that the human-agent relationship should have no gaps — at any level, the agent can pick up the thread. Levels from low to high:

1. **CONVERSATIONAL_CONTINUITY** — within a single session, the discussion stays coherent (ephemeral, handled by context window)
2. **SESSION_CONTINUITY** — across sessions, "where were we?" (`.sessions/`, work-tracker)
3. **PROJECT_CONTINUITY** — across weeks/months, the arc of work, decisions, and direction (`work/`, `docs/`)
4. **SYSTEM_CONTINUITY** — across projects and systems, understanding that a decision here affects something there (future, GRAPH_AGENT territory)
5. **THEMATIC_CONTINUITY** — the highest level. The human's evolving design philosophy, recurring themes, mental models, and ways of thinking that persist across all projects and over years. Not tied to any single system — this is *how the human thinks* about building things. Lives in `docs/` as theme documents, discussion papers, and vocabulary itself. The agent that maintains THEMATIC_CONTINUITY helps the human refine and apply their ideas consistently, even as specific projects come and go.

### SESSION_CONTINUITY
The ability to resume work across sessions — "where were we?", "what was I working on?" The most tangible instance of CONTINUITY. Implemented through session logs, work item state, and offered prompts. The agent reads the last session, scans active work, and presents the human with enough context to re-enter the flow.

### OVERLAPPING_MEMORY
The mechanism that makes CONTINUITY possible. Layered memory from long-term (`docs/` — themes, vocabulary, architecture) through mid-term (`work/` — tickets, decisions, progress) to short-term (`.sessions/` — what happened, what was on your mind) to ephemeral (conversation context). The layers overlap — each one provides context the others can't, and together they give the agent a complete picture.

## Agent Roles

The agent as technical director's assistant — not just a code generator, but an active partner in keeping the human technically grounded.

### COMPRESSOR_AGENT
Distills complex systems into vocabulary the human can carry. Takes sprawling implementation detail and produces the named concepts, terms, and mental models that let a human hold an entire system in their head.

### NARRATOR_AGENT
Maintains the architecture story so the human can follow the system's evolution. Produces and updates ARCHITECTURE_NARRATIVEs — bottom-up stories where each section depends only on what came before.

### IMPLEMENTER_AGENT
Translates the human's compressed instructions into concrete code changes. The agent that unpacks "make this nullable" or "SPLIT_BY_TOKEN after the CURSOR" into dozens of specific edits. Works well when SHARED_VOCABULARY and encoded patterns are in place.

### TEACHER_AGENT
Finds key examples that ground abstract concepts in concrete reality. Produces throwaway code, illustrations, and visual diagrams that make understanding stick. The example *is* the understanding.

### CONTINUITY_AGENT
Preserves decisions and reasoning across sessions. Maintains the institutional memory — not what the code looks like (that's in the code), but *why* it looks that way. Ensures the human can pick up where they left off and that past reasoning isn't lost or contradicted.

### GRAPH_AGENT
(Future, graph-backed.) Surfaces connections across systems that no single codebase view would reveal. An agent working on system A can see that a decision in system B affects it.
