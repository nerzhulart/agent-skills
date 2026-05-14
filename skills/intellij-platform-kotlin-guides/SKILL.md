---
name: intellij-platform-kotlin-guides
description: 'Use when Codex writes, refactors, or reviews IntelliJ Platform / JetBrains IDE plugin Kotlin/Java code. Keywords and triggers: IntelliJ Platform, JetBrains IDE, plugin, Project, Application, Service, project service, application service, CoroutineScope, Disposable, Disposer, parentDisposable, asDisposable, disposeOnCompletion, com.intellij.util.CoroutineScopeKt, childScope, com.intellij.platform.util.coroutines.childScope, MessageBus, DocumentListener, LineStatusTracker, FileEditor, editor, viewer, panel, control, listener subscriptions, lifecycle, memory leak, cleanup, isReleased, AtomicReference, CAS. Pair with kotlin-java-code-guides for generic JVM design rules.'
---

# IntelliJ Platform Kotlin Guides

## Operating Rule

Use this skill for IntelliJ Platform-specific lifetime and subscription decisions. Keep generic Kotlin/Java design guidance in `kotlin-java-code-guides`; use this skill only for platform APIs, platform lifecycle owners, and JetBrains IDE UI/editor/viewer code.

Before non-trivial IntelliJ Platform edits, load the focused references below instead of rediscovering lifecycle rules from scratch.

## Reference Map

- Coroutine scopes, `Disposer` interop, scope-first lifecycle, project/application scope injection, and editor/viewer lifetime: read `references/01-lifetimes-and-coroutines.md`.
- Message bus, document listeners, tracker listeners, and temporary subscription state: read `references/02-subscriptions-and-state.md`.
- Final review checklist for IntelliJ Platform changes: read `references/04-review-checklist.md`.

## Core Directives

- Never use `Project` or `Application` as `Disposable` parents.
- Do not introduce new feature lifecycle based on `Disposable`. Prefer `CoroutineScope`.
- Project/application-wide work belongs in a project/application service with an injected `CoroutineScope`. A private or nested service is fine.
- If a `CoroutineScope` exists, it owns the lifetime. Use `scope.asDisposable()` only for legacy `Disposable` APIs, and never dispose that adapter manually.
- If an existing legacy object is `Disposable`, attach it to the owning scope with `com.intellij.util.CoroutineScopeKt#disposeOnCompletion`. Do not wire scope/disposable cleanup by hand.
- For nested lifetimes, use the extension `com.intellij.platform.util.coroutines.childScope`, called as `parentScope.childScope(...)`, or the local helper.
- Prefer listener APIs with `parentDisposable` or coroutine-native APIs. Avoid manual add/remove pairs.
- Avoid `var` fields, CAS, `AtomicReference`, `isReleased`, and `uninstallCurrent...` for ordinary UI/editor lifecycle.

## Workflow

1. Pick the real owner: service, editor, file editor, viewer, toolwindow content, panel, or control.
2. Use that owner's `CoroutineScope`; create a service if the owner is project/application-wide.
3. Adapt the scope with `asDisposable()` or `disposeOnCompletion` only at legacy API boundaries.
4. Keep subscriptions local and scope-owned.
5. Apply the focused reference checklist before finishing.
