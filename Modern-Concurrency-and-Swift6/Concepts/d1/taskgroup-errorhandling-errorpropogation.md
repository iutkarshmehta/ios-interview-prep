### 1. `Task.isCancelled`

### What it is

`Task.isCancelled` is a **Boolean flag** that tells you:

> “Has this task been asked to stop?”

```swift
if Task.isCancelled {
    return
}
```

Important:

* It does **not** stop the task
* It only reports cancellation
* You decide what to do next

---

### Why it exists

Swift uses **cooperative cancellation**:

* The runtime does not kill tasks
* Tasks must check cancellation and exit themselves

This avoids:

* Resource leaks
* Corrupted state
* Half-executed logic

---

### Typical usage

```swift
func longRunningTask() async {
    for i in 0..<1000 {
        if Task.isCancelled {
            print("Cancelled")
            return
        }
        await doUnitOfWork()
    }
}
```

Here:

* The task stays responsive to cancellation
* Cleanup can happen safely

---

### 2. `Task.checkCancellation()`

### What it is

`Task.checkCancellation()` is the **throwing version** of cancellation checking.

```swift
try Task.checkCancellation()
```

If cancelled:

* Throws `CancellationError`
* Immediately exits normal execution flow

---

### When to use it

Use it when:

* Your function is already `async throws`
* You want cancellation to behave like an error
* You want cancellation to propagate upward automatically

---

### Example

```swift
func loadData() async throws {
    try Task.checkCancellation()
    let data = try await fetchData()
    try Task.checkCancellation()
    process(data)
}
```

This makes cancellation:

* Explicit
* Predictable
* Automatically propagated

---

### Difference Between the Two

| Aspect        | `Task.isCancelled` | `Task.checkCancellation()` |
| ------------- | ------------------ | -------------------------- |
| Throws error  | ❌                  | ✅                          |
| Control style | Manual             | Automatic                  |
| Best for      | Cleanup logic      | Propagation                |

---

### 3. Cancellation Propagation

### Core rule

> **Cancellation flows from parent to child tasks.**

This is possible only because of **task hierarchy**.

---

### Example

```swift
Task {
    async let a = fetchA()
    async let b = fetchB()
    await (a, b)
}
```

If the parent task is cancelled:

* `fetchA` is cancelled
* `fetchB` is cancelled
* No manual wiring required

---

### What does NOT propagate

* Cancellation does NOT propagate to `Task.detached`
* Detached tasks must be cancelled manually

This is why detached tasks are dangerous.

---

### 4. `withTaskGroup`

### What is a Task Group?

A task group lets you:

* Create **multiple child tasks**
* Run them concurrently
* Wait for all results
* Keep everything structured

```swift
await withTaskGroup(of: Int.self) { group in
    group.addTask { computeA() }
    group.addTask { computeB() }
}
```

---

### Key guarantees

* All child tasks belong to the parent
* The group cannot exit until all children finish
* Cancellation propagates automatically

---

### Collecting results

```swift
await withTaskGroup(of: Int.self) { group in
    group.addTask { 1 }
    group.addTask { 2 }

    for await value in group {
        print(value)
    }
}
```

* Results arrive as tasks finish
* Order is not guaranteed

---

### 5. `withThrowingTaskGroup`

### Why it exists

`withTaskGroup` does **not support throwing tasks**.

If child tasks can throw:

* You must use `withThrowingTaskGroup`

---

### Example

```swift
try await withThrowingTaskGroup(of: Int.self) { group in
    group.addTask {
        try await fetchA()
    }

    group.addTask {
        try await fetchB()
    }

    for try await value in group {
        print(value)
    }
}
```

---

### Error behavior (Very Important)

If **any child task throws**:

1. The error is thrown out of the group
2. All remaining child tasks are cancelled
3. The parent task receives the error

This is **fail-fast behavior**.

---

### 6. Task Group Error Handling

### What happens on error

```swift
try await withThrowingTaskGroup(of: Int.self) { group in
    group.addTask { try await taskA() }
    group.addTask { try await taskB() }
}
```

If `taskA` fails:

* `taskB` is cancelled
* Group exits immediately
* Error bubbles up

No partial success unless you design for it.

---

### Designing for partial failure

If you want:

* Best-effort results
* No automatic cancellation

Then:

* Catch errors inside child tasks
* Return `Result` instead of throwing

```swift
group.addTask {
    do {
        return .success(try await taskA())
    } catch {
        return .failure(error)
    }
}
```

---

### Staff-Level Insight (Important)

> **Task groups enforce correctness, not convenience.**

They:

* Prevent leaked tasks
* Prevent silent failures
* Force you to handle cancellation and errors explicitly

---

### Final Mental Model (Memorize This)

* Cancellation is cooperative
* `Task.isCancelled` checks
* `Task.checkCancellation()` throws
* Cancellation flows downward
* Task groups create structured parallelism
* Throwing task groups fail fast
* Detached tasks break propagation

---