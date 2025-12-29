# Swift Concurrency — Foundations Summary

## Lesson 1: Why Concurrency Exists

### 1. The Core Problem
- Apps must remain responsive while doing slow work
- Slow work includes:
  - Network requests
  - Disk I/O
  - Heavy computation
- Waiting on the main thread freezes the UI
- Running work in parallel introduces data races

---

### 2. Concurrency vs Parallelism
- Concurrency: multiple tasks make progress logically at the same time
- Parallelism: tasks execute physically at the same time
- Concurrency is about correctness
- Parallelism is about performance

---

### 3. Pre-Swift Concurrency Approaches
- Threads (`NSThread`)
- Grand Central Dispatch (GCD)
- DispatchQueue, DispatchGroup, Semaphores
- OperationQueue
- Completion-handler based APIs

---

### 4. Why GCD Was Unsafe by Default
- No ownership or lifetime tracking
- No structured task hierarchy
- No automatic cancellation
- Shared mutable state is unprotected
- UI thread correctness is manual
- Deadlocks are easy to create
- Compiler provides no data-race warnings

---

### 5. Problems Caused by GCD
- Callback hell and unreadable flow
- Work outlives its owner
- Silent data races
- UI updates from background threads
- Resource leaks and battery drain

---

### 6. Why Swift Concurrency Was Introduced
- Move concurrency into the language
- Enable compiler-enforced safety
- Provide structured lifetimes
- Enable predictable cancellation
- Prevent data races by default

---

### 7. Structured Concurrency
- Tasks form a tree
- Child tasks are scoped to parents
- Work must finish within its lexical scope
- Cancellation propagates from parent to child
- No runaway background work

---

### 8. Key Mental Shift
- Threads → Tasks
- Queues → async functions
- Callbacks → async / await
- Locks → Actors
- Runtime crashes → Compile-time safety

---

## Lesson 2: async / await — How It Actually Works

### 1. What `async` Means
- Function can suspend execution
- Does not imply background execution
- Suspension points must be explicit

---

### 2. What `await` Means
- Marks a potential suspension point
- Allows the function to pause and resume later
- Required for compiler correctness

---

### 3. Suspension vs Blocking
- Blocking:
  - Thread is occupied
  - Wastes system resources
- Suspension:
  - Thread is released
  - Function state is saved
  - Execution resumes later

---

### 4. Thread Behavior
- `await` does not guarantee thread switching
- Execution may resume on any thread
- Thread identity must never be relied upon
- Tasks are the abstraction, not threads

---

### 5. Compiler Enforcement
- Async functions must be awaited
- Suspension must be acknowledged explicitly
- Async code cannot be called from sync code directly

---

### 6. async Functions as State Machines
- Each async function is transformed into a state machine
- Local variables survive suspension
- Stack does not grow across awaits
- Resume points are managed by the runtime

---

### 7. Error Handling
- Async functions integrate with `throws`
- Errors propagate naturally using `do-catch`
- No nested callbacks for error handling

---

### 8. Bridging Sync and Async Code
- Sync code cannot call async code directly
- `Task {}` creates a new async context
- `Task {}` is fire-and-forget unless awaited

---

### 9. UI Safety
- Thread correctness is not guaranteed by `await`
- `@MainActor` enforces main-thread execution
- Replaces `DispatchQueue.main.async`

---

### 10. Key Rules to Remember
- `async` = function may suspend
- `await` = execution may pause here
- Suspension ≠ blocking
- Threads are an implementation detail
- Tasks define concurrency behavior

---

### 11. One-Line Summary
- Swift Concurrency provides readable, cancellable, and safe concurrency by enforcing correctness at compile time instead of relying on developer discipline.
