---
name: kotlin-java-code-guides
description: Use when Codex is asked to write, refactor, review, or design non-Android JVM Kotlin or Java code and should follow idiomatic Kotlin/Java style, clean API design, architecture boundaries, coroutine/Flow best practices, maintainable implementation shape, and behavior-focused testing. Trigger for Kotlin or Java code changes involving APIs, module/package boundaries, domain modeling, interfaces, coroutine scopes, Flow/StateFlow/SharedFlow, implementation quality, or code review checklists.
---

# Kotlin/Java Code Guides

## Operating Rule

Use this skill as an agent-targeted coding standard for non-Android JVM Kotlin and Java work. Optimize for stable APIs, explicit boundaries, idiomatic language use, testable behavior, and low-coupling implementation.

Before non-trivial Kotlin/Java edits, read `references/README.md` and then load only the referenced files that match the task.

## Reference Map

- Architecture or dependency direction: read `references/01-architecture-boundaries.md`.
- Public/internal API shape: read `references/02-api-design.md`.
- Kotlin style, nullability, sealed types, data classes, DSLs: read `references/03-kotlin-style-and-idioms.md`.
- Coroutines, Flow, scopes, supervision, lifecycle ownership: read `references/07-coroutines-and-flow.md`.
- Java style, records, optionals, builders, exceptions: read `references/04-java-style-and-idioms.md`.
- Implementation complexity, state, errors, naming, decomposition: read `references/05-implementation-quality.md`.
- Tests and review expectations: read `references/06-testing-and-review.md`.
- Final self-review before finishing: read `references/09-agent-review-checklist.md`.
- Source rationale and upstream references: read `references/08-source-map.md` only when provenance matters.

## Core Directives

- Preserve the existing architecture unless the task explicitly includes architecture migration.
- Do not reorganize large existing projects to match a preferred layout; improve boundaries inside the current structure.
- Design the API before implementation. Keep public API small and intentional.
- Prefer domain types over primitive obsession and orphaned framework/reactive primitives.
- Introduce interfaces only for real boundaries, alternatives, or test seams.
- If every implementation of an abstraction must support an operation, put the operation on that abstraction instead of creating a separate one-method capability interface and downcasting.
- Do not introduce pass-through wrappers, dispatchers, or handler holders unless they own a real boundary, lifecycle, policy, or state transition.
- Do not keep classes whose full implementation is delegating one method to another object with the same data; depend on the owning abstraction directly.
- Do not replace a needless wrapper with a function or lambda parameter when a meaningful owner type already exists; inject the owner.
- Avoid fallback paths that hide missing capabilities or broken wiring; prefer explicit contracts and fail-fast errors unless compatibility requires a documented fallback.
- Minimize mutable state: create objects with required data and collaborators up front when possible, prefer `val`/`final` fields, and keep unavoidable mutation private and lifecycle-owned.
- Do not use `AtomicReference`, `lateinit`, nullable slots, or equivalent mutable placeholders to break construction cycles; change the construction order or move the operation to the owning abstraction.
- Keep coroutine work structured: use `coroutineScope` for all-or-nothing operation concurrency, `supervisorScope` for independent siblings with explicit failure policy, and long-lived `CoroutineScope` only for owned lifecycles.
- Avoid `GlobalScope`, hidden private scopes, raw orphaned `Flow`/`Channel` crossing layers, and `suspend` as a default marker for async implementation.
- Keep tests behavior-focused and avoid coupling tests to internal decomposition.

## Workflow

1. Orient in the existing package/module/test structure before editing.
2. Identify the API, ownership, dependency direction, state model, and lifecycle affected by the change.
3. Load the focused reference files from the map above.
4. Implement narrowly using local project conventions first.
5. Run the smallest meaningful verification, then broader checks when shared contracts changed.
6. Apply `references/09-agent-review-checklist.md` before reporting completion.
