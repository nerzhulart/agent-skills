# Source Map

Sources fetched and reviewed on 2026-05-02. This guide pack is an original synthesis for agents; it does not copy source texts beyond small terminology and API names.

## Kotlin

- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html) - baseline code organization, naming, formatting, library-specific conventions.
- [Kotlin Library Authors' Guidelines: Introduction](https://kotlinlang.org/docs/api-guidelines-introduction.html) - problem domain, non-functional criteria, mental complexity, compatibility, documentation.
- [Kotlin API Guidelines: Simplicity](https://kotlinlang.org/docs/api-guidelines-simplicity.html) - explicit API mode, minimal concepts, core API model.
- [Kotlin API Guidelines: Readability](https://kotlinlang.org/docs/api-guidelines-readability.html) - composability, DSLs, extensions, avoiding boolean arguments, numeric type meaning.
- [Kotlin API Guidelines: Consistency](https://kotlinlang.org/docs/api-guidelines-consistency.html) - parameter order, naming, overload behavior, error handling consistency, and tests.
- [Kotlin API Guidelines: Predictability](https://kotlinlang.org/docs/api-guidelines-predictability.html) - defaults, extension points, sealed types, read-only state, validation.
- [Kotlin API Guidelines: Debuggability](https://kotlinlang.org/docs/api-guidelines-debuggability.html) - useful state representation and debugging-oriented API behavior.
- [Kotlin API Guidelines: Testability](https://kotlinlang.org/docs/api-guidelines-testability.html) - avoiding global state and making users' code testable.
- [Kotlin API Guidelines: Backward Compatibility](https://kotlinlang.org/docs/api-guidelines-backward-compatibility.html) - binary/source/behavioral compatibility and API validation.
- [Kotlin API Guidelines: Informative Documentation](https://kotlinlang.org/docs/api-guidelines-informative-documentation.html) - KDoc, examples, documenting inputs, behavior, and exceptions.
- [Kotlin Binary Compatibility Validator](https://github.com/Kotlin/binary-compatibility-validator) - public API dumps and compatibility checks.
- [Effective Kotlin](https://kt.academy/book/effectivekotlin) - practical Kotlin best practices for safety, readability, abstraction design, class design, and efficiency.
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html) - official guide covering coroutine basics, cancellation, dispatchers, composing suspending functions, Flow, exceptions, and shared state.
- [Kotlin Coroutines Basics](https://kotlinlang.org/docs/coroutines-basics.html) - official structured concurrency overview: parent-child lifecycle, cancellation propagation, and dispatcher inheritance.
- [Kotlin Coroutine Context And Dispatchers](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html) - official guide to coroutine context, `Job`, dispatcher selection, parent responsibilities, and custom scope lifecycle.
- [Kotlin Coroutine Exceptions Handling](https://kotlinlang.org/docs/exception-handling.html) - official guide to exception propagation, root coroutine handlers, `SupervisorJob`, and `supervisorScope`.
- [kotlinx.coroutines CoroutineScope API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) - official API contract for scopes, structured concurrency conventions, custom scope creation, and cancellation.
- [kotlinx.coroutines coroutineScope API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) - official semantics for operation-scoped concurrent decomposition and child failure propagation.
- [kotlinx.coroutines supervisorScope API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html) - official semantics for independent child failures under a supervised scope.
- [Kotlin Asynchronous Flow](https://kotlinlang.org/docs/flow.html) - official guide to cold flows, flow context, operators, buffering, cancellation, and collection.
- [kotlinx.coroutines Flow API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/) - API contract for Flow, terminal/intermediate operators, cold and hot streams, `stateIn`, and `shareIn`.
- [Kotlin Coroutines Deep Dive](https://kt.academy/book/coroutines) - practical coroutine and Flow best practices, including Flow, SharedFlow, StateFlow, testing, and common use cases.
- [Kt Academy Constructing a Coroutine Scope](https://kt.academy/article/cc-constructing-scope) - practical guidance on custom background scopes, `SupervisorJob`, dispatchers, exception handlers, and DI-based scope construction.
- [Kt Academy Cancellation in Kotlin Coroutines](https://kt.academy/article/cc-cancellation) - practical guidance on scope cancellation, child cancellation, cancelled scopes, `cancelChildren`, and cooperative cancellation.
- [Kt Academy Exception Handling in Kotlin Coroutines](https://kt.academy/article/cc-exception-handling) - practical guidance on exception propagation, `SupervisorJob`, `supervisorScope`, and supervisor-related pitfalls.
- [Kt Academy Coroutines Use Cases Introduction](https://kt.academy/article/cc-use-cases-intro) - layer-oriented view of coroutine use cases for data/adapters, domain, and presentation/API entry layers.
- [Kt Academy Coroutines Use Cases for Domain Layer](https://kt.academy/article/cc-use-cases-domain-layer) - practical guidance on using `coroutineScope`, `async`, `awaitAll`, `supervisorScope`, timeouts, and Flow processing in domain services.
- [Kt Academy Coroutines Use Cases for Presentation/API/UI Layer](https://kt.academy/article/cc-use-cases-presentation-layer) - practical guidance on entry-layer scope creation, background launches, Flow collection with `launchIn`, and `stateIn`/`shareIn` scope ownership.
- [Kt Academy Coroutines Best Practices](https://kt.academy/article/cc-best-practices) - concise practical guidance such as avoiding immediate `async`/`await` and using suspending functions for single values.
- [When NOT to Use suspend in Kotlin: The Reporting API Anti-Pattern](https://nerzhulart.github.io/blog/suspend-function-antipattern/) - project-local guidance on avoiding `suspend` for fire-and-forget reporting APIs and separating reporting from processing/backpressure.
- [Non-Domain Primitive Orphaning in Kotlin](https://nerzhulart.github.io/blog/primitive-orphaning-kotlin/) - project-local guidance on keeping Flow/Channel ownership attached to domain objects instead of passing orphaned communication primitives through layers.

## Java

- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) - formatting, structure, imports, class member order, braces, naming conventions.
- [Effective Java, 3rd Edition](https://www.pearson.com/en-us/subject-catalog/p/Bloch-Effective-Java-3rd-Edition/P200000000138/9780134686042) - Java API and implementation best practices.
- [Joshua Bloch: How to Design a Good API and Why it Matters](https://research.google/pubs/how-to-design-a-good-api-and-why-it-matters/) - API design principles from Java platform experience.

## Architecture And Review

- [Google Engineering Practices: Code Review](https://google.github.io/eng-practices/review/) - review focus on design, functionality, complexity, and maintainability.
- [Google Engineering Practices: What to look for in a code review](https://google.github.io/eng-practices/review/reviewer/looking-for.html) - design and functionality review heuristics.
- [Google Engineering Practices: Small CLs](https://google.github.io/eng-practices/review/developer/small-cls.html) - small coherent changes and splitting strategy.
- [Google Testing Blog: Test Behavior, Not Implementation](https://testing.googleblog.com/2013/08/testing-on-toilet-test-behavior-not.html) - tests should focus on public behavior.
- [Google Testing Blog: Effective Testing](https://testing.googleblog.com/2014/05/testing-on-toilet-effective-testing.html) - fidelity, resilience, and precision.
- [Google Testing Blog: Test Behaviors, Not Methods](https://testing.googleblog.com/2014/04/testing-on-toilet-test-behaviors-not.html) - separate tests by behavior.

## Explicitly Excluded

Android architecture and Android API guidelines were intentionally excluded because this guide pack targets non-Android Kotlin/Java code.
