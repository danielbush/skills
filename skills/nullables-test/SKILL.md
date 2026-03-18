---
name: nullables-test
description: "Write illustrative tests for code that follows the Nullables pattern. Verifies the class under test is ready (all HARDWIRED_INFRA replaced by INJECTED_INFRA, every dependency has .createNull), then writes narrow, sociable, state-based tests using .createNull(). Tests should illustrate the system's concepts and architecture, not just achieve coverage. Use after applying the nullables-refactor skill, or when writing tests for code that already uses DUAL_FACTORY."
---

# Nullables Test

Write tests for code that follows the Nullables pattern. Tests should illustrate how the system works — grounding the codebase's own concepts through concrete examples.

## Vocabulary

This skill uses terms from `docs/vocabulary.md`. Key terms: HARDWIRED_INFRA, INJECTED_INFRA, DUAL_FACTORY, NULLABLE, CONFIGURABLE_RESPONSE, OUTPUT_TRACKING, INFRASTRUCTURE_WRAPPER, NULLABLE_CLASS, CREATE_BOUNDARY_RULE, VALUE_OBJECT.

## Philosophy

Tests are not just for coverage — they are **illustrations of the system**. A developer reading the tests should learn:

- What the class does and why it exists
- What its key behaviors are, expressed in the system's own vocabulary
- What OUTSIDE_WORLD interactions it has (visible through CONFIGURABLE_RESPONSE and OUTPUT_TRACKING)
- How it handles important edge cases

Write the minimum number of tests that ground these concepts. Don't exhaustively test every path — focus on tests that teach a reader about the system. The system being tested has its own domain, its own vocabulary, its own architecture. The tests should reflect that.

## Precondition: Is This Class Ready To Test?

Before writing tests, verify the class under test is testable. Walk this checklist:

### 1. No HARDWIRED_INFRA remaining

Scan the class for any OUTSIDE_WORLD calls used directly (imported and called inline). If found, the class is not ready — the `nullables-refactor` skill should be applied first.

### 2. Class has DUAL_FACTORY

The class must have `.create()` and `.createNull()`. If not, it's not ready.

### 3. All INJECTED_INFRA has `.createNull()`

For each dependency injected through the constructor:

- Does it have `.createNull()`?
- Does `.createNull()` accept CONFIGURABLE_RESPONSE parameters?
- If not → that dependency needs to be made nullable first.

This may require **recursive refactoring**: follow the dependency graph down to the leaves (INFRASTRUCTURE_WRAPPERs). Every node in the chain must have `.createNull()` for the top-level `.createNull()` to produce a fully nulled instance.

The chain:
```
ClassUnderTest.createNull()
  → calls DependencyA.createNull()
    → calls InfraWrapperX.createNull()  ← leaf, has EMBEDDED_STUB
  → calls DependencyB.createNull()
    → calls InfraWrapperY.createNull()  ← leaf, has EMBEDDED_STUB
```

If any link in this chain is missing `.createNull()`, flag it. Tell the human which dependency needs work and suggest using `nullables-refactor` on it.

### 4. `.createNull()` actually nulls everything

Create an instance via `.createNull()` with no arguments. It should:
- Execute without errors
- Not make any real OUTSIDE_WORLD calls
- Return sensible defaults

If `.createNull()` throws or makes real I/O calls, the EMBEDDED_STUBs or CONFIGURABLE_RESPONSE defaults are incomplete.

## Writing Tests

### What to test

For each class, write tests that illustrate its key behaviors:

#### For INFRASTRUCTURE_WRAPPERs (leaf nodes)

- **Test `.createNull()` works**: creating a NULLABLE instance should succeed with defaults
- **Test CONFIGURABLE_RESPONSE**: configure a specific response and verify the wrapper returns it
- **Test OUTPUT_TRACKING**: if the wrapper has `trackX()`, verify it records interactions
- **Test error paths**: configure error responses via CONFIGURABLE_RESPONSE to verify error handling

These tests verify the wrapper's contract — that the EMBEDDED_STUB faithfully represents what the real system does at the behavior level.

#### For NULLABLE_CLASSes (orchestrators)

- **Test the core behavior**: what does this class *do*? Use `.createNull()` to create an instance, call its methods, verify the result. The test should read like a description of the class's purpose.
- **Test with CONFIGURABLE_RESPONSE**: configure specific dependency responses to test how the class handles different inputs from the outside world
- **Test OUTPUT_TRACKING on dependencies**: verify the class makes the right calls to its dependencies (what was written, in what order) — use trackers, not mocks
- **Test edge cases that matter to the domain**: not every edge case, just the ones that illuminate how the system handles its key scenarios

#### For VALUE_OBJECTs

- Test `.create()` and `.createTestInstance()`
- Test any transformation methods
- Keep it simple — these are usually straightforward

#### For PURE code

- Test with plain inputs and outputs
- No nullable machinery needed

### How to write them

```typescript
describe('ClassName', () => {
  // Use .createNull() — never .create() in unit tests
  // .create() is for narrow integration tests only

  it('describes what the class does in domain terms', () => {
    const instance = ClassName.createNull({
      // CONFIGURABLE_RESPONSE makes the test self-documenting:
      // a reader sees exactly what scenario is being tested
      someResponse: { /* ... */ },
    });

    const result = instance.doSomething();

    expect(result).toEqual(/* expected outcome */);
  });

  it('tracks what was written to the outside world', () => {
    const instance = ClassName.createNull();
    // Get a tracker BEFORE the action
    const tracker = instance.dependency.trackWrites();

    instance.doSomething();

    // State-based assertion — what happened, not how
    expect(tracker.data).toEqual([
      { /* expected write */ },
    ]);
  });
});
```

### What NOT to do

- **Don't use mocks, spies, or stubs** — use `.createNull()` and CONFIGURABLE_RESPONSE instead
- **Don't test implementation details** — test behaviors and outcomes
- **Don't write exhaustive tests** — write illustrative ones that teach a reader about the system
- **Don't use `.create()` in unit tests** — `.create()` is for narrow integration tests that verify real OUTSIDE_WORLD behavior, run separately
- **Don't add tests for things that can't fail** — if the code is PURE and trivial, a test adds noise not signal

## Integration Tests (Sparingly)

For INFRASTRUCTURE_WRAPPERs only, write a small number of narrow integration tests using `.create()`:

- Verify the real wrapper talks to the real external system correctly
- Verify the EMBEDDED_STUB's behavior matches reality
- These go in a separate test directory/command (they're slow, need real services)
- Keep them few — they exist to validate the contract, not to test business logic

## Output

Present the tests with brief commentary explaining what each test illustrates about the system. Group tests by the class they're testing. If any precondition check fails, report what's blocking and what needs to happen first.
