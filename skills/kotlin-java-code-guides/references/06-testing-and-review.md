# Testing And Review

Tests should protect behavior and design, not implementation accidents.

## Test Through Stable Boundaries

Prefer testing public or module-level APIs:

- Domain behavior through domain types.
- Use cases through their input/output contracts.
- Adapters through contract tests or focused integration tests.
- Framework glue through thin smoke tests when needed.

Avoid tests that depend on private helper methods, call order, internal collections, or incidental decomposition.

## Behavior Over Methods

Test one behavior per test. A method with multiple behaviors needs multiple tests. A behavior that spans methods can have one test through the API that expresses the behavior.

Good test names explain the scenario and expected behavior:

```text
rejectsTransferWhenSourceAccountIsClosed
returnsEmptyResultWhenUserHasNoInvoices
persistsEventAfterSuccessfulPayment
```

## Test Fidelity

Use the highest-fidelity dependency that keeps the test small and deterministic:

1. Real implementation.
2. Fake implementation.
3. Stub.
4. Mock.

Mocks are useful for hard-to-trigger failures or verifying an interaction that is the behavior. They are a poor default for ordinary collaborator behavior.

## Test Data

- Keep test data minimal and relevant.
- Prefer named builders or fixtures when setup repeats domain concepts.
- Avoid loops and conditionals in tests unless they directly express a table of cases.
- Do not make tests so DRY that the scenario disappears.
- Assert observable results, not incidental intermediate state.

## Edge Cases

For new or changed behavior, consider:

- Empty input.
- Single element.
- Multiple elements.
- Boundary values.
- Duplicate values.
- Invalid values.
- Nullability boundaries in Java interop.
- Concurrency or repeated-call behavior if relevant.
- External system failure if an adapter is involved.

## Architecture Review

During review, ask:

- Does the change belong in this module?
- Did dependencies continue to point in the intended direction?
- Did public API stay minimal?
- Are internal details hidden?
- Are names domain-specific?
- Is complexity lower or at least contained?
- Are tests focused on behavior?
- Could a future change be made locally, or did this create shotgun surgery?

## Small Changes

Prefer small, self-contained changes:

- Behavior change plus related tests.
- Refactor-only change.
- API introduction plus at least one usage.
- Mechanical formatting or generated changes separate from logic.

Large changes should be decomposed unless decomposition would make the system broken between steps.

