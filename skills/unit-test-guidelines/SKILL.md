---
name: unit-test-guidelines
description: "Classical/Detroit unit test creation and update guidelines with strict guardrails. Use when adding or changing production code, writing or updating unit tests, checking unit test coverage, or deciding if new behavior should be unit-tested."
---

# Unit Test Guidelines

## Core workflow

- Ask: "Can this new/changed behavior be recovered with a unit test?" If yes, proceed. If no, state why and propose a refactor or a higher-level test instead.
- Locate existing unit tests for the behavior.
- If covered, update tests to match the intended external behavior (not implementation). If not covered, add new tests.
- Keep tests small, behavior-focused, and resilient to refactors.

## Core philosophy (Detroit/Classical)

- Validate units of behavior, not units of code.
- Prefer state-based verification over interaction-based verification.
- Use real instances of collaborators unless the dependency is shared or non-deterministic.
- Never mock internal domain objects.

## Absolute guardrails (non-negotiable)

- Never hack a test: do not change test logic/expectations just to force a pass.
- Never change production code solely to satisfy a test if the test reveals a real behavior mismatch.
- Never assert on implementation details (private calls, internal state manipulation, specific execution paths).
- Never use non-determinism in tests (DateTime.Now, Random, Guid.NewGuid, unseeded algorithms).
- Never create shared mutable state between tests (no static/shared fixtures).
- Never use Thread.Sleep or fixed delays for async checks; use async/await or polling with timeouts.
- Never write conditional logic inside a test method (no if/for/while/switch); use parameterized tests.

## FIRST-U quality criteria (all must hold)

- Fast: aim <10ms per test; avoid DB, file I/O, network.
- Isolated: no dependency on other tests; order-independent.
- Repeatable: stable across runs/environments.
- Self-validating: assertions only; no manual inspection.
- Timely: write before or with production code when possible.
- Understandable: intent clear without reading production code.

## Structure (AAA pattern; mandatory)

- Arrange: max 5 lines; use fresh instances. If more, use a test data builder.
- Act: exactly one method call that triggers the behavior.
- Assert: one logical assertion (multiple asserts allowed if checking the same object state).

```ts
test("calculate_RoundNumber_ReturnsMarkup", () => {
  // Arrange
  const calculator = new Calculator();
  const input = 100;

  // Act
  const result = calculator.calculate(input);

  // Assert
  expect(result).toBe(110);
});
```

## Naming convention

Format: MethodName_StateUnderTest_ExpectedBehavior

- MethodName: exact method under test
- StateUnderTest: specific condition or input
- ExpectedBehavior: precise outcome

Examples:
- SaveOrder_NegativeAmount_ThrowsDomainException
- CalculateTax_RetiredCustomer_AppliesZeroPercent

## Test doubles (strict boundaries)

Use doubles ONLY for:
- Shared dependencies: database, filesystem, message queue, external API
- Non-deterministic dependencies: system clock, random generators, environment variables

## Update rules

- If production behavior changes, update/add tests to reflect external behavior.
- If refactoring only, keep tests unchanged unless they assert on implementation details (then fix tests).
- If tests already exist for the behavior, extend them; do not duplicate.
