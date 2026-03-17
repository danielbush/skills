---
status: active
created: 2026-03-17
summary: Decide where each design pattern lives (CLAUDE.md principle vs skill workflow) and draft the CLAUDE.md principles section for deep modules, DI/IoC, consumer-defined interfaces
parent: ../discussion/discussion-driven-development.md
theme: "Theme 2: Design Patterns"
---

## Goal

Separate the design patterns into two layers and write the "always-on" layer.

## The two layers

| Pattern | Layer | Rationale |
|---|---|---|
| A-Frame architecture | Principle (CLAUDE.md) | Structural — applies to all code |
| Deep modules | Principle (CLAUDE.md) | Structural — applies to all code |
| Consumer-defined interfaces | Principle (CLAUDE.md) | Structural — applies to all code |
| Value objects | Principle (CLAUDE.md) | Structural — applies to all code |
| Error mapping (neverthrow) | Principle (CLAUDE.md) | Convention — applies when not using effect-ts |
| Nullables (create/createNull) | Skill (existing) | Has a concrete refactoring workflow |
| DI vs IoC | Needs discussion | Clarify the boundary; effect-ts changes the picture |

## Key question to resolve

**DI vs IoC:** In the nullables pattern, `.create` injects dependencies (DI). But consumer-defined interfaces are IoC — the consumer owns the contract, implementations adapt. These are complementary but the relationship needs to be made explicit. And when effect-ts is in play, it handles both. Where's the boundary?

## Next steps

1. **Extract named concepts from nullables algorithm** — walk through `skills/nullables/SKILL.md` lines 75-158, identify each distinct pattern as an UPPER_SNAKE_CASE entity (DUAL_FACTORY, DELAYED_INSTANTIATION, FACTORY_OBJECT, etc.), build a concept index with pointers back to the algorithm. Let concepts emerge rather than forcing a taxonomy.
2. Draft ~40-line Architectural Principles section for CLAUDE.md (earlier PRD has a good starting draft — see `specs/discussion/architecture-skills-prd.md`)
3. Clarify the DI/IoC/effect-ts boundary in a short discussion
4. Update nullables SKILL.md to reference the principles by name
