# Lifetimes And Coroutines

## Hard Rules

Do not use `Project` or `Application` as `Disposable`.

Bad:

```kotlin
Disposer.register(project, childDisposable)
connection.connect(application)
document.addDocumentListener(listener, project)
```

Good:

```kotlin
document.addDocumentListener(listener, scope.asDisposable())
```

## Prefer Scope Over Disposable

Do not introduce new feature lifecycle based on `Disposable`. Prefer `CoroutineScope`.

Bad:

```kotlin
private val disposable = Disposer.newDisposable("preview")
```

Good:

```kotlin
private val scope: CoroutineScope
```

## Use Services For App/Project Work

- Project-wide work belongs in a project service with an injected `CoroutineScope`.
- Application-wide work belongs in an application service with an injected `CoroutineScope`.
- A service may be private or nested if it is only a feature implementation detail.

## Scope Missing

Do not solve a missing scope by creating a standalone `Disposable`.

Use a service:

```kotlin
@Service(Service.Level.PROJECT)
private class PreviewVcsTrackingService(
    private val project: Project,
    private val scope: CoroutineScope,
) {
    fun createPanelScope(): CoroutineScope =
        scope.childScope("Markdown preview VCS tracking")
}
```

Then pass the shorter scope to the owner:

```kotlin
class PreviewPanelFactory(
    private val service: PreviewVcsTrackingService,
) {
    fun createPanel(document: Document): MarkdownPreviewPanel =
        MarkdownPreviewPanel(service.createPanelScope(), document)
}
```

## Use Shorter Scopes For UI Work

Editor/viewer/panel/control work must use that shorter lifetime, usually via a child scope.

```kotlin
val panelScope = viewerScope.childScope("Markdown preview panel")
return MarkdownPreviewPanel(panelScope, document)
```

## Disposable Interop

Use `Disposable` only at legacy API boundaries. `scope.asDisposable()` is an adapter. Never call `Disposer.dispose()` on it.

Two allowed directions:

- Scope -> `Disposable`: use `scope.asDisposable()`.
- `Disposable` -> scope: use `com.intellij.util.CoroutineScopeKt#disposeOnCompletion`.

This especially matters for old editor/viewer/panel/control APIs. If the old API exposes or requires a `Disposable` object, attach that object to the owner scope with `disposeOnCompletion`; do not manually glue it with `invokeOnCompletion`, `onDispose`, `onClose`, or custom lifecycle flags.

Bad:

```kotlin
class MarkdownPreviewPanel(
    private val project: Project,
    private val document: Document,
) {
    private val subscriptions = Disposer.newDisposable("Markdown preview")

    init {
        Disposer.register(project, subscriptions)
        document.addDocumentListener(listener, subscriptions)
    }
}
```

Good:

```kotlin
class MarkdownPreviewPanel(
    private val scope: CoroutineScope,
    private val document: Document,
) {
    init {
        document.addDocumentListener(listener, scope.asDisposable())
    }
}
```

Bad:

```kotlin
val disposable = LegacyViewerPanel()
scope.coroutineContext.job.invokeOnCompletion {
    Disposer.dispose(disposable)
}
```

Good:

```kotlin
import com.intellij.util.disposeOnCompletion

val disposable = LegacyViewerPanel()
scope.disposeOnCompletion(disposable)
```

## No Parallel Cleanup

Bad:

```kotlin
fun release() {
    view.close()
    Disposer.dispose(scope.asDisposable())
}
```

Good:

```kotlin
class PreviewPanel(
    private val scope: CoroutineScope,
    private val view: PreviewView,
) {
    init {
        installSubscriptions(scope)
    }
}
```

The owner of `scope` ends the lifetime.
