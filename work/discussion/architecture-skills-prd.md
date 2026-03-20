# Architectural Skills — Improvement Plan

Status: **Exploration / Draft**
Last updated: 2026-03-07

---

## The Big Picture

The nullables skill is one piece of a broader **architectural philosophy** for writing code. Other pieces include error handling (neverthrow), deep modules, layer separation, etc. Some of these are **always-on principles** (how code should be structured), others are **actionable workflows** (refactor this, test that).

This creates a natural two-layer approach:

### Layer 1: CLAUDE.md / AGENTS.md — Architectural Principles (Always On)

Concise, authoritative statements about how code should be structured. These apply to *all* code in a project. Think of them as a vocabulary of named concepts that the agent internalizes.

Candidate principles:
- **A-Frame Architecture** — Separate Logic, Infrastructure, Application layers
- **Nullables Pattern** — `.create`/`.createNull` dual factories; embedded stubs; configurable responses
- **Deep Modules** — Prefer classes/modules with simple interfaces but significant implementation depth; avoid shallow wrappers that add complexity without absorbing it
- **Error Handling (neverthrow)** — When not using effect-ts, use `neverthrow` with `ResultAsync`; discriminate errors via `_tag` field; map errors at boundaries
- **Consumer-Defined Interfaces** — The consumer owns the contract; implementations adapt to it
- **Value Objects** — Immutable data classes with `.create` and `.createTestInstance`; no I/O
- **State-Based Testing** — Narrow, sociable tests; verify outputs not interactions; no mocks

These would live in CLAUDE.md (or a referenced file) and be phrased as brief, opinionated guidelines — not full tutorials. The skill provides the depth.

### Layer 2: Skills — Actionable Workflows (On Demand)

Commands that perform specific tasks using the principles above. These trigger when the user asks for help with a concrete action.

The nullables skill remains the home for the detailed *how* — the algorithm, patterns, examples. But it would reference the principles by name, and the principles would reference the skill for depth.

---

## What I Learned from Skill-Creator 2.0

Key patterns from `opensrc/repos/github.com/anthropics/skills/skills/skill-creator/SKILL.md`:

### Writing Style
- **Explain the why**, not just MUSTs. "Today's LLMs are smart. They have good theory of mind."
- **Generalize from examples** — don't overfit to specific test cases
- **Keep the prompt lean** — remove what isn't pulling its weight
- **Use imperative form** in instructions
- **Be "pushy" in descriptions** — undertriggering is the bigger risk than overtriggering

### Skill Structure (Progressive Disclosure)
1. **Metadata** (~100 words) — name + description, always in context, primary trigger
2. **SKILL.md body** (<500 lines ideal) — core instructions, loaded when triggered
3. **Bundled resources** (unlimited) — `references/`, `scripts/`, `agents/`, `assets/`

### Practical Mechanics
- **scripts/** for deterministic tasks (packaging, validation, aggregation)
- **agents/** for specialized subagent instructions (grader, comparator, analyzer)
- **references/** for deep docs, loaded on demand; TOC if >300 lines
- **Domain organization** — when a skill supports multiple variants, organize by variant in references/
- **Eval loop**: draft → test cases → spawn runs → grade → aggregate → viewer → feedback → improve → repeat
- **Description optimization**: separate tooling (`run_loop.py`) to test trigger accuracy

### Key Insight for Us
The skill-creator treats skills as *frameworks for thinking*, not scripts. The goal is to transmit understanding, not just rules. This aligns with putting principles in CLAUDE.md (brief, named, authoritative) and depth in skills (detailed, example-rich, actionable).

---

## Concept Vocabulary

Named concepts that should be consistent across CLAUDE.md and the skill. This is the shared language.

| Concept | Brief Definition | Layer |
|---|---|---|
| **A-Frame Architecture** | Logic / Infrastructure / Application separation | Principle |
| **Dual Factory** | `.create()` / `.createNull()` pair on infra/app classes | Principle |
| **Infrastructure Wrapper** | Class wrapping external I/O; sole purpose is interfacing with outside world | Principle |
| **Embedded Stub** | Fake of a third-party interface inside `.createNull` | Pattern (skill) |
| **Configurable Response** | Named behavior-level params on `.createNull` (not implementation-level) | Pattern (skill) |
| **Output Tracker** | `.trackX()` + EventEmitter for observing side effects in tests | Pattern (skill) |
| **Behavior Simulation** | `.simulateX()` for triggering external events in tests | Pattern (skill) |
| **Logic Sandwich** | Read → Process → Write in application code | Pattern (skill) |
| **Delayed Instantiation** | Passing factory functions instead of instances for lazy creation | Pattern (skill) |
| **Factory Object** | `{ Bar: (...) => Bar.create(...) }` for multiple delayed deps | Pattern (skill) |
| **Cascading Nullables** | `.createNull` calling `.createNull` on all its dependencies | Pattern (skill) |
| **Deep Module** | Simple interface, significant implementation; absorbs complexity | Principle |
| **Consumer-Defined Interface** | Consumer owns the contract; implementations adapt | Principle |
| **Error Mapping** | `mapErr()` at boundaries; `_tag` discriminant; `.cause` for underlying | Principle |
| **Value Object** | Data class, `.create`, `.createTestInstance`; no I/O | Principle |
| **Narrow Sociable Test** | Tests one class with real (nulled) collaborators; state-based | Principle |

**Principle** = belongs in CLAUDE.md as a named guideline
**Pattern (skill)** = detailed implementation in the nullables skill, referenced by name from CLAUDE.md

---

## Proposed CLAUDE.md Structure

The idea: a concise **Architectural Principles** section that names and briefly defines each concept. Not a tutorial — just enough that the agent knows the vocabulary and can apply it. Points to the nullables skill for the full algorithm and patterns.

```markdown
## Architectural Principles

### Code Organization: A-Frame Architecture
Separate code into three layers:
- **Logic**: Pure computation, value objects, no I/O
- **Infrastructure**: Wrappers around external systems (network, disk, DB)
- **Application**: Orchestration; coordinates logic and infrastructure

### Deep Modules
Prefer modules/classes with simple interfaces but significant implementation.
Avoid shallow wrappers that just pass through to another layer.
A class should absorb complexity, not merely redistribute it.

### Nullables Pattern
Infrastructure and application classes use dual static factories:
- `.create()` — production instance, real dependencies
- `.createNull()` — test instance, nulled infrastructure (no I/O)

Constructor receives instantiated dependencies. Tests use `.createNull()`
for fast, sociable, state-based tests without mocks.

See the **nullables skill** for the full implementation algorithm and patterns.

### Error Handling
When not using effect-ts for dependency injection:
- Use `neverthrow` (`ResultAsync`, `Result`) for operations that can fail
- Discriminate errors with a `_tag` field for exhaustive matching
- Map errors at boundaries: infrastructure errors → domain errors via `.mapErr()`
- Attach underlying errors to `.cause`

### Value Objects
- Immutable by default; mutable only for in-memory transformations
- Use `.create()` (may require params) and `.createTestInstance()` (with defaults)
- No I/O operations on value objects

### Testing Philosophy
- **Narrow**: Each test exercises one class/function
- **Sociable**: Use real (nulled) collaborators, not mocks
- **State-based**: Verify outputs and observable state, not interaction patterns
- Track side effects with `.trackX()` / EventEmitter pattern
```

This is ~40 lines. Authoritative, scannable, names the concepts, points to the skill for depth.

---

## Proposed Skill Commands

### `/nullables refactor`
Refactor existing code to use the nullables pattern.
1. Analyze target code — identify layers, direct I/O, mock usage
2. Propose refactoring plan using vocabulary (infrastructure wrapper, dual factory, etc.)
3. Execute step by step, following the algorithm in SKILL.md
4. Migrate tests from mocks to nullables

### `/nullables test`
Write tests for code already using nullables.
1. Read `.create`/`.createNull` signatures
2. Identify configurable responses and trackers
3. Write narrow, sociable, state-based unit tests
4. Suggest integration tests for infrastructure wrappers

### `/nullables audit`
Analyze code for anti-patterns.
1. Scan for `jest.mock`, `vi.mock`, `sinon.stub`, `.toHaveBeenCalled`
2. Find direct I/O not in infrastructure wrappers
3. Flag missing dual factories
4. Produce prioritized refactoring opportunities

### `/nullables wrap`
Create an infrastructure wrapper for an external dependency.
1. Identify the third-party API surface
2. Create class with `.create`/`.createNull` + embedded stub
3. Add configurable responses + output tracker

---

## Structural Reorganization

Current:
```
skills/nullables/
├── SKILL.md (332 lines)
└── references/
    └── nullables-guide.md (606 lines)
```

Proposed:
```
skills/nullables/
├── SKILL.md                    # Trimmed: trigger, overview, command dispatch
├── references/
│   ├── algorithm.md            # The core algorithm (current lines 75-158)
│   ├── patterns.md             # Pattern catalog (infrastructure wrapper, logic sandwich, etc.)
│   ├── vocabulary.md           # Concept definitions (or maybe this lives in CLAUDE.md only?)
│   └── nullables-guide.md      # Existing deep reference (may be absorbed into above)
└── evals/
    └── evals.json              # Test cases for the skill
```

Open question: how much of the current nullables-guide.md is redundant with SKILL.md? They overlap significantly. May want to consolidate.

---

## Open Questions

1. **Scope of CLAUDE.md principles** — Should these be in this repo's CLAUDE.md, or are they meant to be copied into project-specific CLAUDE.md files? (Probably the latter — these are project-level standards.)

2. **Deep modules** — How much detail does this need? It's a well-known concept (Ousterhout) but the agent needs enough to apply it. Maybe 3-5 lines in CLAUDE.md + a brief reference section?

3. **effect-ts boundary** — The current skill mentions "if using effect-ts, use it for injections." How does this interact with the error handling principle? When effect-ts is present, neverthrow isn't needed. This boundary needs to be clear.

4. **Multiple skills vs one skill** — Should error handling (neverthrow) be its own skill? Or is it small enough to just be a CLAUDE.md principle? Probably the latter — it's a principle, not a workflow.

5. **Command implementation** — Should commands be inline in SKILL.md (simpler) or separate agent instruction files (more modular)? Skill-creator uses agents/ for complex subagent tasks. Our commands are simpler — probably inline first.

6. **Consolidating reference material** — SKILL.md and nullables-guide.md overlap heavily. Should we merge into one well-structured reference?

---

## Suggested Order of Work

1. **Draft CLAUDE.md principles section** — Name the concepts, write brief authoritative guidelines. This is the quickest win and establishes the vocabulary.

2. **Consolidate nullables reference material** — Resolve overlap between SKILL.md and nullables-guide.md.

3. **Trim SKILL.md** — Move detailed algorithm and patterns to references. Keep SKILL.md under 300 lines with clear pointers.

4. **Add one command** — Start with `/nullables refactor` as highest-value action.

5. **Manual testing** — Try the command + principles on real code, iterate.

6. **Add remaining commands** — `/nullables test`, then `/nullables audit`.

7. **Formal evals** — Once stable, add eval cases using skill-creator framework.

---

## Session Continuity Notes

### Session 1 (2026-03-07)
- Explored skill-creator 2.0 in depth — captured key patterns above
- Read full nullables SKILL.md (332 lines) and nullables-guide.md (606 lines)
- Identified two-layer approach: CLAUDE.md principles + skill for depth/commands
- Identified broader architectural scope: nullables + neverthrow + deep modules + value objects
- Created this PRD
- **Decision needed:** Start with CLAUDE.md principles draft? Or restructure skill first?
