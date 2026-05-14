# Kotlin/Java Agent Code Guides

This guide pack is written for coding agents working on non-Android JVM Kotlin and Java codebases. It is not a formatting-only style guide. Its purpose is to make agents design stable APIs, preserve boundaries, avoid tightly coupled implementation spaghetti, and leave code easier to test and change.

Use this file as the entry point. Read the referenced files in order when the task involves design, API changes, module boundaries, or non-trivial implementation.

## Read Order

1. [00-agent-operating-protocol.md](00-agent-operating-protocol.md) - how an agent should approach code changes.
2. [01-architecture-boundaries.md](01-architecture-boundaries.md) - modules, layers, dependency direction, and coupling rules.
3. [02-api-design.md](02-api-design.md) - public and internal API design rules.
4. [03-kotlin-style-and-idioms.md](03-kotlin-style-and-idioms.md) - Kotlin-specific idioms and traps.
5. [07-coroutines-and-flow.md](07-coroutines-and-flow.md) - idiomatic coroutine and Flow usage.
6. [04-java-style-and-idioms.md](04-java-style-and-idioms.md) - Java-specific idioms and traps.
7. [05-implementation-quality.md](05-implementation-quality.md) - implementation shape, complexity, error handling, and state.
8. [06-testing-and-review.md](06-testing-and-review.md) - tests, review expectations, and behavior checks.
9. [08-source-map.md](08-source-map.md) - fetched source map and rationale.
10. [09-agent-review-checklist.md](09-agent-review-checklist.md) - final checklist before an agent calls a task done.

## Non-Negotiable Agent Directives

- Design the API before writing the implementation. If the public shape is wrong, clean implementation does not save it.
- Preserve existing architecture unless the task explicitly asks to change it. Extend the local pattern before introducing a new one.
- Keep dependency direction explicit. Domain/core code must not depend on delivery, persistence, framework glue, or generated adapters.
- Do not create cycles between packages, modules, or conceptual layers.
- Keep public API small. Everything public is a contract. Prefer package-private, private, or Kotlin `internal` until exposure is intentional.
- Avoid "manager/service/helper" dumping grounds. A class or function must have one clear reason to change.
- Prefer composition over inheritance unless a real subtype relationship is part of the domain model.
- Introduce interfaces for boundaries, alternatives, or test seams. Do not create one-implementation interfaces as decoration.
- Do not create classes whose only job is forwarding to another handler/callback. Keep the callback at the owner, or introduce a named type only when it owns lifecycle, ordering, synchronization, policy, or a real boundary.
- Do not replace a forwarding class with a function/lambda parameter when the operation belongs to an existing owner type. Depend on that owner type directly.
- Do not add fallback behavior that silently papers over an unsupported capability. Fail fast with a clear contract violation unless the fallback is required for compatibility and has explicit tests.
- Make invalid states hard to represent. Use value types, sealed hierarchies, enums, explicit nullability, and validation at boundaries.
- Minimize mutable state. Prefer fully initialized objects: pass required data and collaborators through constructors or factories, then keep fields/properties immutable unless lifecycle state really must change.
- Keep behavior testable through stable APIs. Do not force tests to reach into internals or mock everything.
- Reject cleverness that reduces local readability. Idiomatic code is clear code, not dense code.

## Agent Work Loop

1. Orient: inspect existing packages, naming, tests, and dependency direction before editing.
2. Shape: write a short mental design brief: API, internal decomposition, dependencies, tests, and migration impact.
3. Implement: keep changes narrow, cohesive, and aligned with existing patterns.
4. Verify: run the smallest meaningful checks first, then broader checks if the touched surface is shared.
5. Review: apply [09-agent-review-checklist.md](09-agent-review-checklist.md) before final output.

## When To Stop And Redesign

Stop coding and redesign the change if any of these becomes true:

- A new method needs boolean flags to select multiple behaviors.
- A new type knows about UI/transport, persistence, and domain rules at the same time.
- A "small" change requires editing many unrelated packages.
- Tests require mocking most collaborators instead of asserting behavior through a stable boundary.
- A public method cannot be documented clearly in two or three sentences.
- You need to say "this is temporary" without a contained migration plan.

## Scope

Included:

- Kotlin and Java JVM application/library code.
- API design, modularity, layering, testability, and code review rules.

Excluded:

- Android-specific architecture and API rules.
- UI design guidance.
- One-off formatting preferences that should be delegated to formatters.
