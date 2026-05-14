# Kotlin Style And Idioms

Kotlin code should be concise because the model is simple, not because logic is compressed.

## Visibility And API

- Default to `private` for implementation details.
- Use `internal` for module-only API.
- Use `public` only when the declaration is a deliberate external contract.
- Specify explicit return types for all public and protected declarations.
- Avoid leaking inferred anonymous, platform, or implementation types from APIs.
- Provide KDoc for public API when behavior is not obvious.

For library-like modules, prefer Kotlin explicit API mode. If the build does not enable it, follow the same rules manually.

## Nullability

- Model absence with nullable types only when absence is part of the domain.
- Eliminate platform types at boundaries by validating or converting them.
- Do not use `!!` except in tiny scopes where a preceding check makes the invariant obvious.
- Prefer early returns or local validation over nested null handling.
- Do not use nullable parameters as mode switches.

```kotlin
fun findUser(id: UserId): User?

fun requireUser(id: UserId): User =
    findUser(id) ?: error("User not found: $id")
```

## Immutability

- Prefer `val` over `var`.
- Treat `var`, `lateinit`, and nullable placeholders as design smells for required state. Use them only when lifecycle or interop makes mutation real.
- When data is available at creation time, pass it through the constructor and keep it as `val` properties instead of assigning it after construction.
- Prefer immutable value objects crossing boundaries.
- Return `List<T>`, `Set<T>`, and `Map<K, V>` unless mutation is part of the contract.
- Keep mutable collections private and expose snapshots or read-only views.
- Use `copy` and new values instead of mutating shared state when practical.

## Data Classes

Use `data class` for plain value carriers. Be careful when exposing data classes as public API:

- `copy`, `componentN`, constructor shape, and property names become part of how callers use the type.
- Do not put complex behavior into a data class that should be a domain object with invariants.
- Keep constructor parameters valid and meaningful.

If invariants are non-trivial, prefer a regular class with factories or validation.

## Sealed Types

Use sealed classes or sealed interfaces for closed sets of variants:

- Result states.
- Domain events inside one bounded context.
- Command variants.
- Protocol messages with known alternatives.

Do not use string constants or integer modes when a sealed type or enum can encode the domain.

## Extension Functions

Use extensions for derived behavior that can be implemented from a type's public contract.

Good uses:

- Convenience operations.
- Domain-specific projections.
- Formatting outside the core model.

Avoid:

- Extensions that need hidden state.
- Member extensions unless the codebase has a strong established reason.
- Extensions that make ownership unclear.

## Scope Functions

Scope functions are not a substitute for naming intermediate concepts.

- Avoid nesting `let`, `run`, `apply`, `also`, and `with`.
- Prefer explicit local variables when the receiver becomes unclear.
- Use `apply` for configuring a newly created object.
- Use `also` for side effects that do not transform the value.
- Use `let` for scoped nullable handling or transformation.
- Use `run` when a block returns a computed result.

If a reader must track several implicit `this` or `it` receivers, rewrite it.

## Functions

- Keep each function at one abstraction level.
- Use expression bodies only when the expression is simple and readable.
- Name functions by effect and result.
- Avoid functions that both query and mutate unless the operation is naturally atomic.
- Prefer top-level functions only for stateless operations that are not owned by a type.

## Coroutines

For coroutine-based code:

- Do not use `GlobalScope` for application work.
- Make blocking operations explicit and isolate them.
- Let callers own lifecycle where possible.
- Keep `suspend` APIs focused on asynchronous work, not as a default style.
- Do not hide fire-and-forget work inside methods without a documented lifecycle and failure policy.

## DSLs

Use a DSL only when callers repeatedly declare structured domain data or configuration.

DSLs must:

- Keep required values as parameters when compile-time guarantees are useful.
- Avoid hidden global state.
- Avoid ambiguous receivers.
- Provide clear defaults.
- Remain smaller than the problem they solve.

## Formatting

Follow Kotlin official coding conventions and let the formatter handle formatting. Do not manually churn whitespace around unrelated code.
