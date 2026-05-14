# Review Checklist

Use this checklist before finishing IntelliJ Platform Kotlin/Java changes.

## Lifetime

- No `Project`/`Application` as `Disposable` parents.
- No new feature lifecycle based on `Disposable`.
- Project/application-wide work lives in a service with injected scope.
- Editor/viewer/panel/control work uses a shorter owner scope.
- Nested work uses `com.intellij.platform.util.coroutines.childScope` (`parentScope.childScope(...)`) or the local helper.
- `scope.asDisposable()` is only an adapter and is never disposed manually.
- Legacy `Disposable` objects are attached to scope with `com.intellij.util.CoroutineScopeKt#disposeOnCompletion`.
- No manual scope/disposable glue via `invokeOnCompletion`, `onDispose`, `onClose`, or lifecycle flags.

## Subscriptions

- Use `parentDisposable` or coroutine-native APIs.
- Keep temporary listeners/trackers local.
- Avoid manual add/remove pairs when parent-owned registration exists.
- Avoid `release`, `close`, `dispose`, `isReleased`, and `uninstallCurrent...` for scope-owned work.
- Prefer a child coroutine with `try/finally` and `awaitCancellation()` over `onClose`/`onDispose`/termination hooks for scope-owned cleanup.
- Avoid CAS/atomics/locks for ordinary UI lifecycle.
