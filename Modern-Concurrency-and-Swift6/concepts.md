**Swift 6.2 Concurrency (2 days)**.

---

## DAY 1 — Core & Structured Concurrency

* Why Swift Concurrency
* Structured concurrency model
* async / await basics
* Suspension vs blocking
* How `await` works
* Continuations
* `withCheckedContinuation`
* `withCheckedThrowingContinuation`
* `Task {}` vs `Task.detached {}`
* Task hierarchy
* Task priorities
* Task-local values
* Cooperative cancellation
* `Task.isCancelled`
* `Task.checkCancellation()`
* Cancellation propagation
* `withTaskGroup`
* `withThrowingTaskGroup`
* Task group error handling
* Parallel task execution
* Actor basics
* Actor isolation
* Actor reentrancy
* `nonisolated`
* Global actors
* `@MainActor`

---

## DAY 2 — Safety, SwiftUI & Advanced Topics

* `Sendable` protocol
* Value types and Sendable
* Reference types and Sendable
* `@unchecked Sendable`
* Cross-actor data access
* Data race detection
* AsyncSequence
* `AsyncStream`
* `AsyncThrowingStream`
* `for await` loops
* SwiftUI `.task`
* `.task` cancellation behavior
* View lifecycle & concurrency
* `@MainActor` ViewModels
* Background work off main actor
* Performance considerations
* Actor contention
* Over-creating tasks
* Deadlocks with actors
* Task priority inversion
* Concurrency debugging tools
* GCD → async/await migration
* Completion handlers → async
* DispatchGroup → TaskGroup
* Combine → AsyncSequence

---

