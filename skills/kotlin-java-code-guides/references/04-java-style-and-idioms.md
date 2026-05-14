# Java Style And Idioms

Java code should make contracts explicit and keep implementation details boring.

## Style Baseline

Follow Google Java Style or the local project style if it is already established.

Default rules:

- One top-level public class per file.
- No wildcard imports.
- Use braces for control flow.
- Keep class members in a logical order that a maintainer can explain.
- Keep overloads together.
- Let `google-java-format` or the project formatter decide layout.

Do not reformat unrelated code while changing behavior.

## Visibility

- Default to package-private for implementation types.
- Use `private` for fields and helpers.
- Use `public` only for intentional API.
- Use `protected` sparingly; it creates subclass contracts that are hard to evolve.
- Avoid exposing fields. Use methods or immutable public final value fields only for simple transparent values where local convention allows it.

## Immutability

Prefer immutable classes:

- Mark classes `final` unless designed for inheritance.
- Make fields `private final`.
- Validate constructor inputs.
- Prefer constructor or factory parameters for all required data and collaborators known at creation time.
- Avoid setter-based required state and non-final fields unless the object's lifecycle genuinely changes after construction.
- Make defensive copies of mutable inputs.
- Return defensive copies or unmodifiable views for mutable internals.

If a class is mutable, document ownership and thread-safety expectations.

## Constructors, Factories, Builders

- Use constructors for simple required state.
- Use static factories when names clarify creation or when returning subtypes/cached instances.
- Use builders for objects with many optional parameters.
- Do not create builders for two required fields.
- Keep builders separate from the immutable value they build.

## Interfaces And Classes

- Prefer interfaces for capabilities and boundaries.
- Prefer classes for state and behavior with invariants.
- Prefer composition over inheritance.
- Design inheritance explicitly or prohibit it with `final`.
- Do not make a class extensible unless you can document and test subclass contracts.

## Generics

- Use generics to express real type relationships.
- Avoid raw types.
- Use bounded wildcards for producer/consumer APIs where they improve usability.
- Keep type parameters meaningful and consistently named.
- Do not expose generic complexity that callers do not need.

## Nullness

Java needs explicit nullness discipline:

- Annotate APIs when the codebase has a nullness annotation convention.
- Reject `null` at boundaries unless it is part of the contract.
- Use `Objects.requireNonNull` for required constructor and method arguments.
- Return empty collections instead of `null` collections.
- Use `Optional<T>` for return values where absence is expected.

Do not use `Optional` for fields, collection elements, or parameters by default.

## Exceptions

- Throw `IllegalArgumentException` for invalid caller arguments.
- Throw `IllegalStateException` for invalid receiver state.
- Preserve causes when wrapping exceptions.
- Do not swallow exceptions.
- Do not catch broad exceptions unless the boundary can translate them meaningfully.
- Use try-with-resources for closeable resources.

## Records And Sealed Types

Use records for shallow immutable data carriers when:

- The state is the API.
- Invariants are simple and can be enforced in the compact constructor.
- Generated equality and string representation match domain expectations.

Use sealed classes/interfaces for closed hierarchies when the supported variants are part of the contract.

## Streams

Streams should improve clarity:

- Prefer streams for simple transformations and aggregations.
- Prefer loops for complex branching, mutation, checked exceptions, or debugging-heavy logic.
- Avoid nested streams that hide control flow.
- Name intermediate results when the pipeline becomes dense.
