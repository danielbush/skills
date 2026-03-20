---
status: active
id: NULLABLES_SKILLS__WORK
created: 2026-03-18
summary: All work related to /nullables-refactor, /nullables-test, and /explain-nullables skills
parent: ../../docs/discussion-driven-development.md
theme: "Theme 2: Design Patterns"
---

# Nullables Skills

Covers the `/nullables-refactor`, `/nullables-test`, and `/explain-nullables` skills — building, evaluating, and improving them.

## Done

- [x] Created `nullables-refactor` skill (`skills/nullables-refactor/SKILL.md`) — side-effect boundary classification, HARDWIRED_INFRA detection, CREATE_BOUNDARY_RULE checks
- [x] Ran evals on nullables-refactor: with-skill 95.8% vs without-skill 88.5% (4 fixtures, 26 assertions)
- [x] Created `nullables-test` skill (`skills/nullables-test/SKILL.md`) — precondition checks, recursive nullability verification, illustrative test philosophy
- [x] Clarified test instantiation: `new Foo(Bar.createNull())` when testing Foo itself vs `Foo.createNull()` when Foo is a dependency (confirmed per James Shore)
- [x] Created `docs/vocabulary.md` — 15+ terms including HARDWIRED_INFRA, INJECTED_INFRA, CREATE_BOUNDARY_RULE, NULLABLE_CLASS
- [x] Vocabulary sharing via symlinks so both skills reference one source
- [x] Moved original monolithic nullables skill to `sandbox/nullables-skill/`

## Outstanding

### Evals for nullables-test
Create fixture files and eval suite for the test skill, similar to what was done for nullables-refactor (4 fixtures, 26 assertions).

### Improve nullables-refactor skill
Eval gap: skill misses noting compliant code (e.g. `findUser()` correctly using INJECTED_INFRA via `this.db`). Add guidance to call out what's already working, not just violations.

### Incorporate testing principles into nullables-test
Encode these into the test skill's guidance:
1. **AAA pattern** — Arrange/Act/Assert; the Act step should directly call the thing under test, not a helper (within reason)
2. **Test level** — prefer middle or top-down testing over exhaustive bottom-up; avoid overlapping tests at slightly different levels that exercise almost the same paths; let coverage guide where additional tests are needed
3. **Instantiation** — `new Foo(dep.createNull())` when testing Foo itself (fine-grained CONFIGURABLE_RESPONSE control); `Foo.createNull()` when Foo is a dependency (already in SKILL.md line 69+, reinforce)

### /explain-nullables skill
Standalone skill that reads vocabulary + refactor/test SKILL.md files as references and explains the framework at whatever level is asked:
- Full walkthrough (side-effect boundaries → HARDWIRED vs INJECTED → WRAPPER vs NULLABLE_CLASS → refactor→test pipeline)
- Zoom into a specific term with examples
- Explain how the pattern applies to a specific file in the user's codebase

Starting point for a broader `/explain` capability.

### Feedback from real usage
Human may try skills on another project (jsed is the primary testbed). Incorporate findings and adjust skill guidance.
