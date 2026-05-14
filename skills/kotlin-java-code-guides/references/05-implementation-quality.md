# Implementation Quality

Implementation quality is mostly about controlling complexity, state, and dependency knowledge.

## One Level Of Abstraction

Each function should operate at one conceptual level.

Bad shape:

```text
parse request -> validate domain -> query database -> apply policy -> format response
```

Better shape:

```text
handler: translate request and response
use case: coordinate operation
domain: enforce policy
repository/adapter: load and store
```

Extraction is useful when the extracted name is a domain concept or a stable step, not when it merely hides lines.

## Complexity Tripwires

These are not universal hard limits. They are signals that an agent must pause and simplify:

- More than two nested control-flow levels.
- A method that needs comments to explain phases.
- A method that touches many unrelated fields.
- A class with several unrelated groups of methods.
- An interface with many methods not needed by most implementations.
- A function with several nullable or boolean parameters.
- Repeated conditionals checking the same mode or type.

Prefer:

- Guard clauses.
- Small named domain operations.
- Polymorphism or sealed variants for real behavior differences.
- Value objects for repeated parameter groups.
- Strategy objects only when behavior truly varies.

Do not extract a class only because a callback, handler, method reference, or object reference needs to be passed around. Extraction must name ownership, lifecycle, synchronization, policy, or a reusable domain operation.

## Naming

Names should identify domain concepts and responsibilities.

Avoid generic names unless the local codebase uses them with clear meaning:

- `Manager`
- `Helper`
- `Util`
- `Processor`
- `Handler`
- `Service`
- `Impl`
- `Base`

If a name needs one of these words, ask what capability or policy it represents.

## State And Side Effects

- Keep mutation local.
- Before adding mutable fields/properties, check whether the value can be local, constructor-supplied, or represented by an immutable value object.
- Avoid partially initialized objects. Prefer passing all required data into the constructor or factory, then store it in immutable fields/properties.
- If mutable state exists only to forward to another handler, keep it in the lifecycle owner or remove it by passing the final handler directly.
- If a class only maps one method call to another method call with the same data, remove the class and depend on the owning abstraction directly.
- Do not convert a one-method wrapper into a one-method function parameter if a named collaborator already owns that operation. That preserves indirection without improving design.
- Do not use `AtomicReference`, `lateinit`, nullable slots, arrays, or other mutable placeholders to break dependency construction cycles. If two objects cannot be built without a placeholder, fix the API shape, construction order, or owner boundary.
- Do not wrap immutable dependencies in `AtomicReference`, `volatile`, or nullable mutable slots just to model close/invalid state. Put lifecycle state in the owner that actually accepts or rejects operations.
- Make side effects visible in names or API placement.
- Do not mix queries and commands unless the domain operation is naturally atomic.
- Do not hide writes behind getters or computed properties.
- Do not let constructors perform IO or start background work.
- Do not cache unless there is a measured or obvious need and an invalidation policy.

## Dependency Injection

Use constructor injection for required collaborators.
Required data belongs there too when it is known at construction time; do not hide it behind late setters or `initialize` methods.

Avoid:

- Service locators.
- Static mutable collaborators.
- Hidden global clocks, random generators, clients, or executors.
- Optional dependencies that are actually required in some modes.

If a dependency is optional, model the optional behavior explicitly.

## External Systems

Do not let vendor or transport types invade core code.

Translate at boundaries:

- HTTP/request DTOs to command/input values.
- Database rows/entities to domain/application values.
- Vendor errors to domain/application errors.
- External timestamps/IDs to local value types.

Keep adapters boring and testable.

## Error Messages

Error messages should help the caller fix the problem:

- Include safe relevant values.
- Name the violated contract.
- Preserve original causes.
- Avoid leaking secrets or sensitive data.
- Keep internal exception types out of public API unless they are intentional.

Fallbacks are error-handling policy, not harmless convenience. Add one only when the caller can still get correct behavior; otherwise fail fast and name the missing capability or violated contract.

## Concurrency

- Prefer immutable shared data.
- Keep locks private and minimal.
- Document thread-safety for public mutable types.
- Avoid calling external code while holding locks.
- Use structured concurrency for async work.
- Make cancellation and failure propagation explicit.
- Use atomics for real concurrent state transitions or lock-free coordination, not as a default holder for values that are assigned once.

## Comments

Use comments for why, invariants, non-obvious tradeoffs, and external constraints.

Do not comment what the code already says. If a comment is needed to explain normal control flow, first try to simplify or rename the code.
