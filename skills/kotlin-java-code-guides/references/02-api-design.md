# API Design

An API is any surface another class, package, module, service, or user depends on. Public language visibility is only one kind of API. Internal module boundaries and test fixtures are also APIs.

## Design From The Problem Domain

Before adding an API, define:

- What problem it solves.
- Who calls it.
- What concepts callers must understand.
- Which state it owns.
- Which errors it can produce.
- How it can evolve.

If the domain words are unclear, do not invent generic names like `Processor`, `Handler`, `Manager`, or `Helper`. Find the actual capability.

## Small Conceptual Model

Prefer a small set of core operations and build convenience operations on top.

Good APIs have:

- Few concepts.
- Predictable names.
- Consistent parameter order.
- Clear ownership of state.
- Explicit failure modes.
- Extension points where variation is expected.
- Closed types where variation is invalid.

Bad APIs have:

- Boolean flags controlling unrelated behavior.
- Repeated groups of primitive parameters.
- Methods that accept or return framework/vendor types from the wrong layer.
- Names that describe implementation instead of capability.
- Wrapper types that only forward to another handler without owning policy, lifecycle, synchronization, or a boundary.
- Fallback behavior that hides an unsupported required capability instead of making the contract explicit.
- Public types that exist only because the implementation needed them.

## Public Surface Rules

- Public API must be intentionally public.
- Public functions and properties must have explicit return types.
- Public API must not expose mutable implementation state.
- Public API must not expose internal package types.
- Public API must document non-obvious behavior, invalid inputs, side effects, threading, and failure modes.
- Public API must have at least one usage in production or tests when introduced.

For Kotlin library-style modules, enable explicit API mode or follow it manually. For Java, use package-private visibility by default and expose only what callers need.

## Type Design

Use types to prevent misuse:

- Use value classes, records, or small value objects for IDs, money, units, names, and domain-specific primitives.
- Use enums when the set is stable and closed.
- Use sealed classes/interfaces when variants are closed but carry different data.
- Use interfaces for capabilities and boundaries.
- Use concrete immutable classes for values.
- Use collections by interface type: `List`, `Set`, `Map`, not concrete implementations.

If an operation is mandatory for all implementations of an abstraction, declare it on that abstraction. Do not create a separate one-method capability interface that callers must discover with `as?`, `instanceof`, or fallback behavior.

If a caller needs one operation from an object that already owns the relevant lifecycle and invariants, accept that object. Do not hide the ownership behind a one-method adapter, function type, or lambda parameter unless the function itself is the domain concept.

Avoid primitive obsession:

```kotlin
// Weak: both Long values can be swapped.
fun transfer(fromAccountId: Long, toAccountId: Long, cents: Long)

// Stronger: domain roles are explicit.
fun transfer(from: AccountId, to: AccountId, amount: Money)
```

Avoid primitive orphaning:

- Do not pass a raw primitive, collection, `Flow`, `Channel`, callback, or framework handle through layers when a domain object owns its meaning and lifecycle.
- Preserve the owner type when it carries context, invariants, lifecycle, permissions, or related operations.
- Use raw primitives and framework types freely inside narrow implementation scopes where ownership stays obvious.

```kotlin
// Weak: delivery mechanism is detached from its owner.
fun process(events: Flow<Event>)

// Stronger: ownership and semantics stay discoverable.
fun process(eventBus: EventBus)
```

## Parameters

- Put required parameters before optional parameters.
- Keep parameter order consistent across related functions.
- Avoid same-typed parameter runs where callers can swap arguments.
- Avoid boolean mode parameters. Prefer named operations or enum modes.
- Avoid nullable parameters as behavior switches.
- Use a request/options value type when parameters form a concept.

```java
// Weak: the boolean is unclear at call sites.
report.generate(userId, true);

// Better: behavior is named.
report.generateIncludingArchived(userId);
```

## Mutability

- Accept read-only views when mutation is not required.
- Return read-only collections or defensive copies.
- Do not return arrays from public API unless mutation and copying semantics are explicit.
- Prefer APIs that construct complete objects: required data and collaborators should be constructor or factory parameters, not late setters, nullable placeholders, or mutation-only setup calls.
- If a factory needs a mutable placeholder to wire two collaborators together, the API is in the wrong shape. Build the owner first, then connect the dependent through an explicit owner method, or move the operation onto the owner.
- Keep builders separate from immutable built values.
- Do not let callers mutate internal caches or state.

Do not expose a type only to store a mutable callback or delegate. Either make the owner expose the receiving operation directly, or introduce a boundary type with a meaningful contract.

## Error Handling

Choose one error model per API family:

- Return nullable/optional when absence is expected and not exceptional.
- Return a result type when failure is expected and carries useful data.
- Throw exceptions when the caller violated a contract or the operation cannot proceed.
- Do not use exceptions for normal control flow.
- Validate inputs at the boundary with clear error messages.

Kotlin:

- Use `require` for invalid caller arguments.
- Use `check` for invalid object state.
- Use nullable return types for expected absence.
- Use `Result` carefully in public API because it affects composition and language interop.

Java:

- Use `Optional<T>` for return values where absence is expected.
- Do not use `Optional` for fields or parameters by default.
- Use checked exceptions only when callers can reasonably recover and the codebase convention supports them.

## Compatibility

For public or cross-module APIs, treat these as breaking unless explicitly managed:

- Renaming packages, classes, methods, parameters used from Kotlin named calls, or enum constants.
- Changing return types, nullability, generic bounds, or exception behavior.
- Adding abstract methods to interfaces.
- Removing constructors or changing default values.
- Changing equality, ordering, serialization, or threading semantics.

Prefer additive evolution:

- Add a new method with a clearer name.
- Deprecate old API with replacement guidance.
- Keep binary-compatible shims where needed.
- Use compatibility validators for libraries.

## Documentation

Document contracts, not syntax.

Good API documentation says:

- What the operation means.
- What inputs are valid.
- What is returned.
- What side effects happen.
- What errors can occur.
- Whether the operation is thread-safe, blocking, idempotent, cached, or expensive.
- A minimal example for non-obvious usage.

Do not restate the signature in prose. If documentation cannot be written clearly, the API is probably wrong.
