# Coroutines And Flow

This guide is for idiomatic non-Android Kotlin coroutine and Flow code. Treat coroutines as structured work with explicit lifetimes, not as a prettier thread API. Treat Flow as an asynchronous stream contract, not as a default wrapper for every value.

## Core Model

- A `suspend` function represents one asynchronous result or operation.
- A `Flow<T>` represents a stream of zero or more values over time.
- A `StateFlow<T>` represents observable state with a current value.
- A `SharedFlow<T>` represents shared events or broadcasts.
- A `CoroutineScope` owns launched work and defines its lifetime.
- A `Job` is a low-level coroutine primitive. Do not expose it from business APIs; use `CoroutineScope` ownership or suspending operations to control work.

## Choose The Right Shape

Use `suspend fun` when the caller expects one result:

```kotlin
suspend fun loadUser(id: UserId): User
```

Use `Flow<T>` when the caller expects a sequence of updates:

```kotlin
fun observeUser(id: UserId): Flow<User>
```

Do not return `Flow<T>` just because the implementation is asynchronous. A flow implies repeated or streaming values. If the operation naturally has one result, use a suspending function.

## When Suspend Is The Contract

Use `suspend` when the caller must participate in the operation:

- The caller needs the result before continuing.
- The caller needs to know when the operation completed.
- Backpressure is intentional and callers should slow down when the callee cannot keep up.
- The operation crosses a real suspending boundary such as non-blocking IO, a delayed resource wait, or coroutine coordination.

Do not mark a function `suspend` just because the implementation uses coroutines somewhere below it. A suspending signature is a caller-visible contract: it says the caller may wait, cancellation may propagate, and backpressure may be introduced.

Prefer a regular function for fire-and-forget reporting APIs:

```kotlin
interface ChangeCollector {
    fun report(change: FileChange)
}
```

If processing is asynchronous, let the owning component batch, debounce, conflate, or launch that work under its own explicit lifecycle. Do not force every reporter to wait for downstream processing unless waiting is the intended behavior.

Use a separate suspending operation when the caller explicitly asks for processing:

```kotlin
interface ChangeCollector {
    fun report(change: FileChange)
    suspend fun processPending(): ProcessingResult
}
```

For `MutableStateFlow`, prefer `value = ...` or `update { ... }` for state updates. Do not use `emit` to make a simple state assignment look like meaningful asynchronous work.

## Structured Concurrency

- Launch coroutines in a scope owned by the caller, component, request, or application lifecycle.
- Prefer `coroutineScope { ... }` when a suspending function needs child coroutines.
- Prefer `supervisorScope { ... }` only when sibling failure independence is part of the contract.
- Avoid `GlobalScope`. It opts out of structured concurrency and usually creates work that outlives its owner.
- Do not create a new long-lived scope inside ordinary classes unless that class explicitly owns a lifecycle and exposes `close`, `cancel`, or equivalent cleanup.

Good concurrent composition:

```kotlin
suspend fun loadProfile(id: UserId): Profile = coroutineScope {
    val user = async { users.load(id) }
    val settings = async { settings.load(id) }
    Profile(user.await(), settings.await())
}
```

Do not use `async { ... }.await()` immediately. If no work overlaps, call the suspend function directly or use `withContext` for a dispatcher change.

## Scope Selection And Use Cases

Choosing a `CoroutineScope` means choosing when launched work is cancelled, where uncaught failures are reported, and which dispatcher/context children inherit.

Use these defaults:

- In a `suspend` function, use `coroutineScope { ... }` for concurrent decomposition of one operation.
- Use `supervisorScope { ... }` only when child tasks are intentionally independent and each child handles or reports its own failure.
- Use a long-lived `CoroutineScope` only in an object that owns a lifecycle longer than one suspending call.
- Prefer framework/request scopes when the framework already provides them. Do not wrap every request in an extra custom scope.
- Pass a scope only to components that launch work, own shared hot flows, or adapt callback/background APIs. Do not thread `CoroutineScope` through domain APIs as a convenience parameter.

### Normal Scope Vs Supervisor Scope

Use normal structured concurrency for all-or-nothing work:

```kotlin
suspend fun loadDashboard(userId: UserId): Dashboard = coroutineScope {
    val profile = async { profileRepository.load(userId) }
    val permissions = async { permissionsRepository.load(userId) }
    Dashboard(profile.await(), permissions.await())
}
```

If any child fails, `coroutineScope` fails, cancels siblings, and rethrows to the caller. This is correct for one logical operation where partial success is invalid.

Use supervision for independent sibling work:

```kotlin
suspend fun publishAuditEvents(events: List<AuditEvent>) = supervisorScope {
    events.forEach { event ->
        launch {
            runCatching { auditSink.publish(event) }
                .onFailure { failure -> logger.warn("Audit publish failed", failure) }
        }
    }
}
```

If one child fails in `supervisorScope`, siblings continue. Parent cancellation still cancels all children. If the `supervisorScope` block itself throws, its children are cancelled. Each supervised child needs a local failure policy because child failure is no longer automatically turned into whole-operation failure.

Use `SupervisorJob()` only in infrastructure or composition-root code that constructs an owned long-lived scope:

```kotlin
val applicationScope = CoroutineScope(
    SupervisorJob() + Dispatchers.IO + exceptionHandler
)
```

Do not use `SupervisorJob()` as a local builder argument to "make this launch supervised". It is easy to detach work from the intended parent and it does not give child coroutines inside the launched block the semantics people usually expect. Use `supervisorScope { ... }` for local supervised decomposition.

Do not use `withContext(SupervisorJob())` as a replacement for `supervisorScope`. `withContext` is for changing context of one logical operation; `supervisorScope` is the scope function that changes child-failure semantics.

### Organizing Scope Ownership

A scope should be owned by one lifecycle:

- Request or command lifecycle: usually provided by the caller/framework; use `suspend` functions and `coroutineScope`.
- Operation lifecycle inside a `suspend` function: use `coroutineScope`, `supervisorScope`, `withTimeout`, or `withContext`.
- Component lifecycle: pass in or create one scope for the component.
- Application/process lifecycle: create one named application scope at composition root.
- Test lifecycle: use a test scope and inject it where background work is launched.

If a class creates a scope, it owns cleanup. If a class receives a scope, the provider owns cleanup. Do not cancel a scope you did not create unless the ownership contract says so.

If no suitable scope is immediately available, do not fall back to an unrelated owner, global owner, or ad-hoc `close()` protocol. Find the lifecycle that semantically owns the feature or activity and use that scope. If the lifecycle belongs to a session, screen, request, job, component, or service, inject that scope from the corresponding owner or composition root.

When a nested activity needs its own lifetime, make that owner explicit and pass in the scope that belongs to it. Do not invent generic machinery just to manufacture lifetimes. Framework-specific scope helpers belong in framework-specific guidance.

```kotlin
class ReportScreen(
    private val scope: CoroutineScope,
    private val pipeline: ReportPipeline
) {
    fun start() {
        pipeline.startIn(scope)
    }
}
```

The important part is ownership: `ReportScreen` does not create a lifetime out of thin air. The composition root, UI framework, request framework, or service that owns the screen provides the scope.

Prefer constructor injection for scopes that are part of the environment:

```kotlin
class CacheRefreshService(
    private val repository: CacheRepository,
    private val applicationScope: CoroutineScope
) {
    fun refresh() {
        applicationScope.launch {
            repository.refresh()
        }
    }
}
```

This service does not expose `close()` because it does not own `applicationScope`. The composition root or framework cancels the scope during application shutdown.

When an infrastructure adapter creates its own private scope, it must expose cleanup:

```kotlin
class PollingWorker(
    private val client: RemoteClient,
    dispatcher: CoroutineDispatcher,
    exceptionHandler: CoroutineExceptionHandler
) : AutoCloseable {
    private val scope = CoroutineScope(
        SupervisorJob() + dispatcher + exceptionHandler
    )

    fun start() {
        scope.launch {
            while (isActive) {
                client.poll()
                delay(POLL_INTERVAL)
            }
        }
    }

    override fun close() {
        scope.cancel()
    }
}
```

This shape is acceptable for adapters that really own resources. It is not a reason to put `Job` or private scopes into ordinary feature or business services.

### Scopes Instead Of Parallel Release Protocols

Prefer scope-owned lifetimes over creating new manual `close` or `release` protocols in feature code. A separate lifecycle tree is often an older interop mechanism; it should not become the primary lifetime model when a coroutine scope already exists.

If an older callback API requires a lifecycle owner, registration owner, or cancellation token, derive it from the owning scope when the framework provides such an adapter. Do not manually end the derived owner; doing so duplicates ownership and can invalidate siblings that also rely on the same scope-owned parent.

Bad: inventing a parallel owner and cleanup path:

```kotlin
class ReportPanel(
    private val events: EventSource,
    private val listener: EventListener
) {
    private val owner = ManualLifecycleOwner("report-panel")

    fun start(scope: CoroutineScope) {
        events.addListener(listener, owner)
        scope.launch { collectUpdates() }
    }

    fun release() {
        owner.close()
    }
}
```

Better: bind callback APIs to the scope that already owns the activity:

```kotlin
class ReportPanel(
    private val events: EventSource,
    private val listener: EventListener,
    private val scope: CoroutineScope,
    private val lifecycleAdapters: LifecycleAdapters
) {
    init {
        val registrationOwner = lifecycleAdapters.ownerFor(scope)
        events.addListener(listener, registrationOwner)
    }
}
```

The derived registration owner is only an adapter for legacy listener APIs. The scope remains the owner. Synchronous listener registration in a constructor or factory is acceptable when the scope-owned parent handles unsubscription. Do not hide coroutine work in `init`; start launched work from a factory, `startIn(scope)`, or another caller-owned setup point whose ordering and failure policy are explicit.

If the object receives a scope, do not close resources that the scope-created API already owns:

```kotlin
class ReportPanel(
    private val view: ScopeOwnedView
) {
    fun close() {
        view.close() // smell when createView(scope) already makes scope the owner
    }
}
```

End the owner scope instead. Keep explicit `close`, `release`, or `use` blocks only for resources whose API really requires deterministic cleanup and is not already owned by the scope.

### Cleanup Coroutine Instead Of Lifecycle Hooks

When a `CoroutineScope` owns the lifetime, do cleanup inside a child coroutine. Prefer `try/finally` around real work, or `awaitCancellation()` when the coroutine exists only to bind a resource to the scope.

Bad:

```kotlin
val subscription = events.subscribe(listener)
scope.coroutineContext.job.invokeOnCompletion {
    subscription.close()
}
```

Good:

```kotlin
scope.launch {
    val subscription = events.subscribe(listener)
    try {
        awaitCancellation()
    }
    finally {
        subscription.close()
    }
}
```

For active work:

```kotlin
scope.launch {
    val resource = openResource()
    try {
        process(resource)
    }
    finally {
        resource.close()
    }
}
```

Use `onClose`, `onDispose`, `invokeOnCompletion`/`invokeOnTermination`, or similar hooks only at API adapter boundaries where the API itself forces that shape.

### Scopes Instead Of Initialize And Close

Prefer binding background work to a caller-provided scope over creating lifecycle methods that must be called in the right order.

Do not start coroutine work from constructors or `init` blocks. Construction should establish required state, not launch asynchronous behavior that callers cannot await, order, or handle. If setup must suspend, expose a suspending factory or suspending initialization operation and call it from the caller-owned coroutine. If the object owns a long-lived listener, collector, or debounce pipeline, start that work from an explicit `startIn(scope)`/factory step using the caller-owned scope rather than hiding `scope.launch` in `init`.

Use a `startIn` or `launchIn` style API when the object describes work but should not own the lifecycle. Do not return `Job` from business-level APIs; cancellation should come from the owning scope.

```kotlin
class IndexingPipeline(
    private val events: Flow<FileEvent>,
    private val index: SearchIndex
) {
    fun startIn(scope: CoroutineScope) {
        scope.launch {
            events.collect { event ->
                index.apply(event)
            }
        }
    }
}
```

Callers then bind the pipeline to the correct lifetime:

```kotlin
class Application(
    private val scope: CoroutineScope,
    private val indexingPipeline: IndexingPipeline
) {
    fun start() {
        indexingPipeline.startIn(scope)
    }
}
```

This avoids hidden `initialize()` ordering and keeps cancellation on the owner: cancelling `scope` stops the work.

For work that can be naturally represented as data, prefer cold `Flow` over start/close lifecycle:

```kotlin
class ChangeWatcher(
    private val fileSystem: FileSystem
) {
    fun changes(): Flow<FileChange> = callbackFlow {
        val subscription = fileSystem.watch { change ->
            trySend(change).isSuccess
        }
        awaitClose { subscription.close() }
    }
}
```

A cold flow starts when collected and stops when collection is cancelled. That is often cleaner than `initialize()`, listener registration, and `close()`.

Do not replace real resource ownership with scopes blindly. Files, sockets, thread pools, native handles, and external subscriptions still need deterministic cleanup. A scope can own the coroutine lifecycle around that resource, but the resource itself still needs a clear close path, usually in `finally`, `use`, or an adapter-level `close`.

Shared hot flows:

```kotlin
class LocationService(
    locationDao: LocationDao,
    applicationScope: CoroutineScope
) {
    private val locations = locationDao.observeLocations()
        .shareIn(
            scope = applicationScope,
            started = SharingStarted.WhileSubscribed(),
            replay = 1
        )

    fun observeLocations(): Flow<List<Location>> = locations
}
```

The scope passed to `shareIn` or `stateIn` owns upstream collection. Choose a request scope for request-local sharing, a component scope for component-local caching, or an application scope for process-level sharing.

Testing:

- Prefer coroutine test utilities such as `runTest` when available.
- Inject scopes or dispatchers into components that launch background work.
- Cancel or advance test-owned scopes explicitly so tests do not depend on real time or leaked background work.

## Dispatcher Ownership

- Do not hardcode dispatchers deep in domain logic.
- Use `withContext` at the boundary where blocking or CPU-heavy work is known.
- Use `Dispatchers.IO` for blocking IO.
- Use `Dispatchers.Default` for CPU-bound work.
- Keep dispatcher choices injectable for reusable libraries or code that needs deterministic tests.
- Do not emit from a different context inside `flow {}`. Use `flowOn` to move upstream execution.

## Cancellation

Cancellation is part of the API contract.

- Make long-running loops cooperative with suspending calls, `yield`, or `ensureActive`.
- Do not swallow `CancellationException`.
- If catching broad exceptions, rethrow cancellation.
- Use `withTimeout` only when timeout is a real business or resource boundary.
- Release resources in `finally`, using non-cancellable cleanup only when suspension during cleanup is required.

```kotlin
catch (e: CancellationException) {
    throw e
} catch (e: Exception) {
    // translate or handle real failure
}
```

## Exceptions

- `launch` is for fire-and-report work owned by a scope.
- `async` is for concurrent work whose result will be awaited.
- Exceptions in child coroutines should normally cancel the parent operation.
- Use `supervisorScope` when one child failure should not cancel siblings.
- Keep `CoroutineExceptionHandler` for top-level reporting, not local business recovery.

## Flow Design

Flows are cold by default. Building a flow should be cheap; work starts when a terminal operator collects it.

- Keep flow builders side-effect-light until collection.
- Prefer operator chains over manual collection and re-emission.
- Use `map`, `filter`, `transform`, `scan`, `combine`, and `flatMap*` to describe stream semantics.
- Keep intermediate values immutable, especially with `scan` and shared flows.
- Avoid hidden launches in flow-returning functions.
- Do not expose `MutableStateFlow` or `MutableSharedFlow`; expose read-only `StateFlow`, `SharedFlow`, or `Flow`.

## Flow Context

- Flow collection runs in the collector's coroutine context by default.
- Use `flowOn` to change upstream execution context.
- Do not use `withContext` around `emit` inside `flow {}`.
- Use `buffer` only when producer and consumer should run concurrently.
- Use `conflate` when the newest value matters more than every intermediate value.
- Use `collectLatest` when new input should cancel processing of older input.

## Hot Flows

Use `StateFlow` for state:

- It always has a current value.
- New collectors receive the latest value.
- It should represent state, not one-off events.

Use `SharedFlow` for events:

- Configure `replay` intentionally.
- Keep replay small unless historical events are part of the contract.
- Prefer exposing `SharedFlow<T>` and keeping `MutableSharedFlow<T>` private.

Use `stateIn` or `shareIn` when converting a cold upstream flow into a shared hot stream. The provided scope then owns the upstream collection, so choose that scope deliberately.

## Avoid Stream Orphaning

`Flow`, `StateFlow`, `SharedFlow`, `ReceiveChannel`, and `Channel` are communication primitives. They are not domain owners by themselves.

Avoid passing raw streams through multiple architectural layers when a domain object owns the stream, its lifecycle, or its semantics. Pass the owner instead:

```kotlin
class EventBus(
    val events: SharedFlow<Event>
)

class EventProcessor(
    private val eventBus: EventBus
)
```

This keeps ownership, lifecycle, naming, and related operations discoverable. A parameter like `EventBus` tells the reader who owns the events. A parameter like `Flow<Event>` only says how values are delivered.

Raw stream parameters are acceptable for private helpers, local transformations, framework utilities, and direct producer-consumer handoffs where ownership remains obvious. They are a smell when they cross module, layer, or domain boundaries and the reader must trace backwards to discover who created them.

Treat nested reactive types such as `Flow<Flow<T>>` or `StateFlow<Flow<T>>` as design pressure. They often mean the outer stream represents changing ownership, changing sources, or a domain relationship that should be modeled explicitly.

## Channels

Prefer Flow for streams exposed as API. Use channels for communication between coroutines when send/receive semantics, fan-out, fan-in, or backpressure are central to the implementation.

Do not expose channels as public API unless callers really need channel semantics.

## API Guidelines

- Name one-shot APIs with verbs: `loadUser`, `sendCommand`, `calculatePrice`.
- Name stream APIs as observations: `observeUser`, `updates`, `events`, `states`.
- Return `Flow<T>` for cold streams and `StateFlow<T>` or `SharedFlow<T>` only when hotness is part of the contract.
- Keep mutability private: `_state: MutableStateFlow<T>` and `state: StateFlow<T>`.
- Use `asStateFlow()` and `asSharedFlow()` when exposing read-only views from mutable flows.
- Keep stream owners explicit. Prefer passing a domain owner over passing its raw `Flow` or `Channel` through unrelated layers.
- Document whether a flow is cold or hot when it is not obvious.
- Document cancellation, retry, timeout, buffering, and replay behavior when they affect callers.

## Testing

- Test suspending functions as behavior, not implementation scheduling.
- Avoid sleeps in tests; use coroutine test utilities when the project has them.
- Test cancellation for long-running or resource-owning operations.
- Test Flow with a bounded collection: `first`, `single`, `take(n).toList()`, or test utilities.
- For hot flows, test initial value, replay behavior, and lifecycle of upstream collection when relevant.

## Common Anti-Patterns

- `GlobalScope.launch { ... }` inside library or domain code.
- Creating a private scope just to avoid making a caller-facing operation `suspend`.
- Passing `CoroutineScope` through domain APIs that do not own or launch lifecycle-bound work.
- Creating `CoroutineScope(Job())` for a long-lived component where sibling failures should be isolated.
- Using `withContext(SupervisorJob())` as a replacement for `supervisorScope`.
- `runBlocking` inside suspend functions or production request paths.
- `async { work() }.await()` with no overlapping work.
- Marking fire-and-forget reporting APIs as `suspend`.
- Returning `Flow<T>` for a single value.
- Exposing `MutableStateFlow` from a class.
- Passing orphaned `Flow` or `Channel` values through layers without their domain owner.
- Using nested reactive types when an explicit owner or source object would preserve context.
- Emitting from `withContext` inside `flow {}`.
- Catching `Exception` and accidentally swallowing cancellation.
- Creating a new `CoroutineScope` per method call without ownership or cancellation.
- Using `StateFlow` for one-off events.
- Adding `buffer`, `conflate`, or `flatMapMerge` without a semantic reason.
