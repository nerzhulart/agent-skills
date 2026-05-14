# Agent Operating Protocol

This protocol turns the guide pack into agent behavior. Follow it before and during code changes.

## Prime Directive

Make the smallest coherent change that preserves or improves the codebase structure. A coherent change has a clear owner, clear boundary, clear API, clear tests, and no hidden coupling.

## Before Editing

Answer these questions from the existing code, not from guesses:

- Which package, module, or layer owns this behavior?
- What is the current dependency direction?
- What are the stable APIs around this area?
- Which types are public contracts and which are implementation details?
- Which tests describe the behavior today?
- Is the requested change feature work, bug fix, refactor, or API evolution?

If the codebase already has a pattern for the same problem, prefer that pattern unless it is clearly the source of the problem.

## Design Brief For Non-Trivial Changes

Before coding a change that adds new behavior, new public API, new cross-package calls, or new state, form this brief:

```text
Problem:
User-facing behavior:
Public or module API:
Internal decomposition:
Dependency direction:
State ownership:
Error handling:
Testing strategy:
Compatibility risk:
```

The brief can be mental for small work. For larger work, write it in the task response or a local design note if the user asked for a plan.

## Editing Rules

- Put new behavior at the lowest layer that owns the rule.
- Keep framework glue thin. It should translate inputs, call the core, and translate outputs.
- Keep persistence code about persistence. It should not decide domain policy.
- Keep domain/core code free from transport, database, filesystem, clock, random, and environment dependencies unless those are passed through ports or explicit collaborators.
- Keep constructors honest. Required dependencies and required state must be constructor parameters.
- Prefer fully initialized objects over `initialize`/setter phases. If data is known at creation time, pass it through the constructor or factory and store it immutably.
- Do not add pass-through holder objects to make wiring look cleaner. If a holder is unavoidable, name the lifecycle or synchronization policy it owns; otherwise keep the reference local to the owner.
- Do not replace pass-through objects with anonymous function plumbing when a stable collaborator already owns the operation. Constructor-inject the collaborator.
- Do not add permissive fallback code for a missing interface/capability. Treat missing required capabilities as wiring errors unless the task explicitly requires compatibility behavior.
- Avoid ambient dependencies: global state, static mutable registries, hidden singletons, and service locators.
- Keep refactors separate from behavior changes when they are large enough to distract review.

## Interface Rules

Introduce an interface only when at least one of these is true:

- It defines a boundary between modules or layers.
- It has multiple production implementations now or a known imminent second implementation.
- It is a port to external infrastructure.
- It lets tests use a faithful fake without exposing internals.

Do not introduce `FooInterface`, `IFoo`, or `FooService` only because the implementation is named `FooServiceImpl`. If the interface is useful, name it after the domain capability. If only the class exists, name the class well and keep it concrete.

## Coupling Control

When a change seems to require importing a higher layer into a lower layer, do not do it. Instead:

- Move the rule down to the owning layer.
- Pass required data as a value type.
- Add a port interface in the core and implement it outside.
- Split the operation into separate application and infrastructure steps.

## Agent Failure Modes To Avoid

- Creating a single large method that performs parsing, validation, business rules, persistence, and output formatting.
- Adding a new service class as a dumping ground for unrelated workflow steps.
- Solving every variation with booleans, nullable parameters, or string modes.
- Mocking internals because the design has no clean boundary.
- Adding public methods "just in case".
- Duplicating architecture vocabulary without enforcing the dependency rules behind it.

## Done Means

A task is done only when:

- The change is placed in the right owner.
- Public API is minimal and documented when appropriate.
- Dependencies still point in the intended direction.
- Complex logic is decomposed by domain concepts, not by arbitrary line count.
- Behavior is covered by meaningful tests or the absence of tests is explicitly reported.
- Static checks or relevant tests have been run when feasible.
