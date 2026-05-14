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
- Are atomics/volatile fields used only for real concurrent state transitions, not for immutable collaborators or misplaced lifecycle guards?
- Did the change avoid `AtomicReference`/`lateinit`/nullable placeholders whose only purpose is to break construction order?
- Are external resources closed and failures preserved?
- Is concurrency lifecycle, cancellation, or locking explicit where relevant?

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
