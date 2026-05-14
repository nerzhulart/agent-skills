# Agent Review Checklist

Run this checklist before finalizing a Kotlin/Java coding task.

## Architecture

- Does the change belong to the package/module where it was placed?
- Do dependencies still point in the intended direction?
- Did the change avoid package/module cycles?
- Are framework, persistence, transport, and vendor details kept out of core code?
- Are internals hidden behind module API?

## API

- Is every public declaration intentional?
- Can the API be explained clearly without describing implementation internals?
- Are names domain-specific and consistent with nearby code?
- Are parameter order, nullability, mutability, and failure modes explicit?
- Are mandatory operations declared on the owning abstraction rather than discovered through capability downcasts?
- Did the change avoid pass-through wrapper types that only forward to another handler/callback?
- Are required capabilities expressed as contracts instead of hidden fallback behavior?
- Did the change avoid boolean mode flags and nullable behavior switches?
- Can required state be supplied up front through constructors or factories instead of post-construction mutation?
- Did the change avoid exposing mutable internal state?

## Implementation

- Does each class/function have one clear responsibility?
- Is complex logic split by domain concepts rather than arbitrary helpers?
- Does every extracted class own lifecycle, policy, synchronization, a boundary, or a reusable operation?
- Did the change avoid classes that only delegate one method to another object with the same input?
- Did the change avoid replacing a needless wrapper with anonymous function plumbing when a named owner type exists?
- Are side effects visible and owned by the right layer?
- Is state ownership clear?
- Is mutable state minimal, private, and justified by real lifecycle needs?
- Does every new `var` have a clear owner, lifetime, threading story, and invalidation rule?
- Could any mutable field be a local variable, constructor value, immutable value replacement, or explicit observable state instead?
- Is business/UI state that other code reacts to represented by an explicit reactive model such as `StateFlow` rather than scattered mutable fields?
- Are temporary listener/subscription details kept local to the setup/cleanup point instead of being promoted to object fields?
- Are atomics/volatile fields used only for real concurrent state transitions, not for immutable collaborators or misplaced lifecycle guards?
- Did the change avoid `AtomicReference`/`lateinit`/nullable placeholders whose only purpose is to break construction order?
- Did the change avoid manual locking/CAS/synchronization in feature or business code unless it is implementing a real concurrency primitive?
- Are external resources closed and failures preserved?
- Is concurrency lifecycle, cancellation, or locking explicit where relevant?
- If a scope is provided, did the code avoid inventing a parallel close/release lifecycle?
- If an older callback API requires a lifecycle owner or cancellation token, is it derived from the semantic scope and left owned by that scope?
- Did the change avoid manually closing/cancelling/discarding resources already owned by a scope passed to their factory?
- For scope-owned cleanup, did the code prefer a child coroutine with `try/finally` and `awaitCancellation()` over `onClose`/`onDispose`/termination hooks?
- Did business APIs avoid exposing `Job` and use `CoroutineScope` ownership or suspending operations for lifecycle control?

## Kotlin

- Are public/protected return types explicit?
- Is visibility minimized?
- Are platform types contained at boundaries?
- Are scope functions readable and not nested into unclear receiver chains?
- Are sealed/value/data classes used only where their generated API matches the contract?

## Java

- Is visibility minimized?
- Are mutable inputs and outputs defensively handled?
- Are null contracts explicit?
- Are exceptions specific and useful?
- Are inheritance points deliberate and documented, or classes final?

## Tests

- Do tests assert behavior through stable APIs?
- Is each test focused on one behavior?
- Are edge cases covered for new logic?
- Are fakes preferred over mocks when they give better fidelity?
- Are architecture rules tested when the change affects module boundaries?

## Verification

- Did the agent run the smallest meaningful test/check?
- If checks were not run, is the reason explicit?
- Did relevant project checks run when available?
- Did the agent avoid unrelated whitespace and formatting churn?
