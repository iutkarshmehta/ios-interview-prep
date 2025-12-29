

### 1. Task Hierarchy

### What Is Task Hierarchy?

In Swift Concurrency, tasks are organized into a **tree**.

* A task can create child tasks
* Child tasks belong to their parent
* Parent tasks control the lifetime of children

This is **structured concurrency** in action.

---

### Why Task Hierarchy Exists

Without hierarchy:

* Tasks run independently
* Cancellation doesn’t propagate
* Lifetime is unpredictable

With hierarchy:

* Ownership is clear
* Cancellation is automatic
* Errors propagate correctly

---

### Example

```swift
Task {
    async let a = fetchA()
    async let b = fetchB()
    await (a, b)
}
```

* The outer `Task` is the parent
* `fetchA` and `fetchB` are children
* Parent cannot finish until children finish

---

### Cancellation in Hierarchy

If the parent task is cancelled:

* All child tasks are cancelled automatically
* No manual wiring needed

This is impossible without a hierarchy.

---

### 2. Task Priorities

### What Is Task Priority?

Task priority expresses **how important a task is**, not how fast it must run.

Priorities influence **scheduling**, not correctness.

---

### Available Priorities

```swift
.userInitiated
.userInteractive
.utility
.background
```

* Higher priority → scheduled sooner
* Lower priority → runs opportunistically

---

### Priority Inheritance

Child tasks inherit priority from their parent:

```swift
Task(priority: .userInitiated) {
    Task {
        // Also userInitiated
    }
}
```

This prevents:

* Priority inversion
* Starvation of important work

---

### Priority Is a Hint, Not a Guarantee

* Tasks may not run immediately
* System load matters
* Priorities are advisory

Never rely on priority for correctness.

---

### 3. Task-Local Values

### What Are Task-Local Values?

Task-local values are:

* Values scoped to a task
* Automatically inherited by child tasks
* Invisible outside the task tree

Think of them as **async-safe context storage**.

---

### Why Task-Local Values Exist

Thread-local storage does not work in async code because:

* Threads change across `await`
* Tasks move between threads

Task-local values solve this.

---

### Example

```swift
enum RequestIDKey {
    @TaskLocal static var value: String?
}
```

Setting a value:

```swift
RequestIDKey.$value.withValue("123") {
    await performWork()
}
```

Reading it:

```swift
print(RequestIDKey.value) // "123"
```

---

### Inheritance Rules

* Child tasks inherit task-local values
* Detached tasks do NOT inherit them
* Values are immutable unless overridden

---

### Common Use Cases

* Request IDs
* Logging context
* Tracing
* Debug metadata

---

### 4. Cooperative Cancellation

### What Is Cooperative Cancellation?

Swift cancellation is **cooperative**, not forced.

* Tasks are not killed
* Tasks must **check for cancellation**
* Cancellation is a signal, not an interrupt

---

### How Cancellation Is Triggered

* Parent task is cancelled
* Explicit `task.cancel()` call
* SwiftUI view disappears
* Task group error

---

### How Tasks Observe Cancellation

```swift
if Task.isCancelled {
    return
}
```

Or:

```swift
try Task.checkCancellation()
```

---

### Cancellation at `await`

Many async APIs:

* Check cancellation automatically
* Throw `CancellationError`

Example:

```swift
try await Task.sleep(nanoseconds: 1_000_000_000)
```

Cancels immediately if cancelled.

---

### Cleanup on Cancellation

Use `defer`:

```swift
Task {
    defer {
        cleanup()
    }
    try await longTask()
}
```

---

### Cancellation Propagation

* Parent → child (automatic)
* Child → parent (via thrown error)
* Detached tasks → no propagation

---

### Staff-Level Insight

> Cancellation only works because tasks are structured.

Without hierarchy:

* Cancellation becomes manual
* Bugs reappear

---

### Final Mental Model

* Tasks form a tree
* Parents own children
* Priority flows downward
* Context flows via task-locals
* Cancellation flows downward cooperatively

---

