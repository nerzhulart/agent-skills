# Architecture Boundaries

Good architecture is not a diagram. It is a set of dependency rules that the code actually follows.

## Core Rules

- Dependencies must point from outer details to inner policy, not the other way around.
- Domain/core code owns business rules and must not know delivery mechanisms, persistence details, or framework lifecycle.
- Application/use-case code coordinates domain operations and ports. It should not become a second domain model.
- Infrastructure adapts external systems. It implements ports; it does not define business policy.
- Public module API is the only legal entry point from other modules.
- Internal packages are internal even if language visibility makes them technically importable.

## Package And Module Shape

For new or nearly new projects, prefer a package/module layout that makes boundaries visible:

```text
feature/
  api/          stable types used by other modules
  internal/     implementation details
  domain/       core model and rules, if the feature owns domain logic
  application/  use cases and orchestration
  infra/        adapters for persistence, network, filesystem, vendors
```

This is a shape, not a mandate. Apply it only when the project is new enough, or when there is explicit agreement that refactoring toward a new layout is in scope.

In existing projects, especially large ones, do not reorganize packages or modules just to match this layout. Preserve the local structure, improve boundaries inside the current shape, and make only the smallest layout changes needed for the task.

## Dependency Direction

Allowed direction in a layered design:

```text
delivery/framework -> application -> domain
infrastructure -> application/domain ports
domain -> domain only
```

Forbidden direction:

```text
domain -> infrastructure
domain -> delivery/framework
domain -> application service orchestration
application -> delivery/controller/view
```

If a lower layer needs something from an outer layer, define a port in the lower layer and implement it outside.

## API And Internal Packages

- Treat each module's public API as a product.
- Put implementation details behind an `internal` package or Kotlin `internal` visibility.
- Do not import another module's `internal` package.
- Do not expose implementation class names, storage schemas, HTTP models, database entities, or vendor DTOs through module API.
- Do not name public types `Impl`, `Base`, `Helper`, or `Util` unless the name is truly part of an established local pattern.

## Cycles

Cycles are architecture debt, even when the compiler accepts them.

Do not create cycles between:

- Gradle/Maven modules.
- Java packages.
- Kotlin packages.
- Feature modules.
- Domain aggregates.
- Service classes.

Break cycles by extracting a smaller domain concept, moving a rule to its owner, or introducing a port at the dependency inversion point.

## Ports And Adapters

Use a port when core/application logic needs a capability supplied by the outside world:

- Current time.
- Randomness or ID generation.
- Persistence.
- Remote service calls.
- File or process IO.
- Message publishing.
- Secrets and environment.

A port should speak the domain/application language, not the vendor language. For example, prefer `InvoiceRepository.save(invoice)` over `JdbcInvoiceGateway.executeInsert(row)`.

## Abstraction Budget

Abstractions are not free. Add one when it buys at least one of these:

- Enforces a boundary.
- Names a stable domain concept.
- Removes meaningful duplication.
- Allows independent replacement of an external dependency.
- Makes behavior testable without exposing internals.

Do not add an abstraction only to mirror a pattern from another architecture article.

## State Ownership

- First ask whether mutable state is needed at all. Prefer immutable values supplied through constructors or factories when the data is known up front.
- Each piece of mutable state must have one owner.
- Shared mutable state must be rare, explicit, synchronized when needed, and hidden behind a narrow API.
- Prefer immutable values crossing boundaries.
- Prefer read-only collections in APIs.
- Do not expose mutable internal collections, arrays, builders, or caches.

## Framework Boundaries

Framework annotations and lifecycle concerns should stay near the outside:

- Controllers, listeners, jobs, and CLI commands translate external input into application calls.
- Persistence adapters translate domain/application values into storage operations.
- Dependency injection configuration wires components; it does not contain business logic.

If core code is hard to instantiate without a framework container, the boundary is probably wrong.
