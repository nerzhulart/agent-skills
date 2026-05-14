# Subscriptions And State

## Prefer Parent Disposable Over Add/Remove Pairs

For IntelliJ listener APIs, prefer overloads that accept a `parentDisposable`. When the owner is a coroutine scope, use `scope.asDisposable()` at the registration point.

Bad:

```kotlin
class PreviewPanel(
    private val document: Document
) {
    private val documentListener = object : DocumentListener {
        override fun documentChanged(event: DocumentEvent) {
            scheduleUpdate()
        }
    }

    fun start() {
        document.addDocumentListener(documentListener)
    }

    fun release() {
        document.removeDocumentListener(documentListener)
    }
}
```

Better:

```kotlin
class PreviewPanel(
    private val scope: CoroutineScope,
    private val document: Document
) {
    init {
        val listener = object : DocumentListener {
            override fun documentChanged(event: DocumentEvent) {
                scheduleUpdate()
            }
        }

        document.addDocumentListener(listener, scope.asDisposable())
    }
}
```

The listener is a local implementation detail. It does not need a field if no other method uses it.

## Keep Subscriptions Local

Avoid fields such as `listener`, `connection`, `subscriptionDisposable`, `currentSubscription`, `isReleased`, and helpers such as `unsubscribeCurrent()` when a listener, message bus connection, tracker, document listener, or UI callback can be modeled as one scope-owned registration.

Bad:

```kotlin
class PreviewPanel {
    private var currentSubscription: Subscription? = null
    private var listener: ChangeListener? = null

    private fun install(source: ChangeSource?) {
        currentSubscription?.removeListener(listener)
        currentSubscription = source
        listener = ChangeListener { scheduleUpdate() }
        source?.addListener(listener!!)
    }

    private fun unsubscribeCurrent() {
        currentSubscription?.removeListener(listener)
        currentSubscription = null
        listener = null
    }
}
```

Better:

```kotlin
private fun installChangeSubscription(
    scope: CoroutineScope,
    source: ChangeSource,
) {
    val owner = scope.asDisposable()
    val listener = object : ChangeListener {
        override fun changed() {
            scheduleUpdate()
        }
    }

    source.addListener(listener, owner)
}
```

If the exact API offers a Flow or another coroutine-native subscription, prefer that over listener interop.

## Message Bus

Use a scope-derived `Disposable` for message bus connections that are scoped to a panel/editor/viewer.

```kotlin
private fun subscribeToTopics(
    scope: CoroutineScope,
    project: Project,
) {
    val connection = project.messageBus.connect(scope.asDisposable())
    connection.subscribe(MyTopic, object : MyListener {
        override fun changed() {
            scheduleUpdate()
        }
    })
}
```

Do not keep `MessageBusConnection` in a field only to disconnect it later. Let the parent disposable disconnect it.

## Avoid Defensive Lifecycle Noise

Do not wrap ordinary listener registration in broad `try/catch` blocks to compensate for unclear lifecycle. Do not add CAS operations, `AtomicBoolean`, or `isReleased` guards around normal UI subscription code. If registration can race with disposal, fix the owner scope and registration ordering.
