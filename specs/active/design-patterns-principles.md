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

1. Draft ~40-line Architectural Principles section for CLAUDE.md (PRD.md has a good starting draft)
2. Clarify the DI/IoC/effect-ts boundary in a short discussion
3. Update nullables SKILL.md to reference the principles by name
