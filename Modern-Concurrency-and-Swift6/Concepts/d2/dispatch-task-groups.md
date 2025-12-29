### DispatchGroup → TaskGroup (Swift Concurrency)

### Why DispatchGroup Existed (First Principles)

DispatchGroup was created to:

* Wait for multiple asynchronous tasks
* Notify when all tasks finish

But it has **no understanding of ownership, cancellation, or errors**.

Core problem:

> **DispatchGroup tracks completion, not task lifetimes.**

---

### Why TaskGroup Replaces It

TaskGroup is part of **structured concurrency**:

* Tasks have a parent
* Cancellation propagates
* Errors are typed
* Lifetimes are scoped

Key shift:

> **From “counting callbacks” → “owning concurrent work”**

---

### Mental Model Shift (Critical)

| DispatchGroup      | TaskGroup               |
| ------------------ | ----------------------- |
| Manual enter/leave | Automatic lifecycle     |
| Global queues      | Structured tasks        |
| No cancellation    | Cancellation propagates |
| Error-unaware      | Typed errors            |
| Easy to leak       | Scoped & safe           |

---

### Step-by-Step Migration Strategy

1. Identify DispatchGroup usage
2. Determine if tasks are static or dynamic
3. Choose `async let` or `TaskGroup`
4. Convert callback APIs to async
5. Handle errors explicitly
6. Move UI updates to `MainActor`
7. Remove `enter/leave` logic

---

### Step 1: Identify DispatchGroup Patterns

Typical code:

```swift
let group = DispatchGroup()

group.enter()
fetchA { group.leave() }

group.enter()
fetchB { group.leave() }

group.notify(queue: .main) {
    updateUI()
}
```

Problems:

* Forgetting `leave()` causes deadlock
* No error handling
* No cancellation
* Hard to reason about ownership

---

### Step 2: Static Tasks → `async let` (Preferred)

If you know the number of tasks at compile time.

### DispatchGroup Version

```swift
group.enter()
fetchUser { user in
    group.leave()
}

group.enter()
fetchPosts { posts in
    group.leave()
}

group.notify(queue: .main) {
    updateUI()
}
```

---

### async let Version

```swift
async let user = fetchUser()
async let posts = fetchPosts()

let (u, p) = await (user, posts)
updateUI()
```

Why better:

* No manual tracking
* Cancellation propagates
* Errors bubble automatically

Rule:

> **Use `async let` when tasks are fixed in number.**

---

### Step 3: Dynamic Tasks → TaskGroup

If tasks depend on runtime data.

### DispatchGroup Version

```swift
let group = DispatchGroup()
var results: [Data] = []

for url in urls {
    group.enter()
    fetch(url) { data in
        results.append(data)
        group.leave()
    }
}

group.notify(queue: .main) {
    updateUI(results)
}
```

Problems:

* Data race on `results`
* Manual lifecycle
* No error handling

---

### TaskGroup Version

```swift
let results = await withTaskGroup(of: Data.self) { group in
    var collected: [Data] = []

    for url in urls {
        group.addTask {
            await fetch(url)
        }
    }

    for await data in group {
        collected.append(data)
    }

    return collected
}

updateUI(results)
```

Key benefits:

* No shared mutable state
* Safe iteration
* Automatic cleanup

---

### Step 4: Handling Errors (Big Upgrade)

DispatchGroup ignores errors.

### TaskGroup with Errors

```swift
let results = try await withThrowingTaskGroup(of: Data.self) { group in
    for url in urls {
        group.addTask {
            try await fetch(url)
        }
    }

    var collected: [Data] = []
    for try await data in group {
        collected.append(data)
    }

    return collected
}
```

If one task fails:

* Remaining tasks are cancelled
* Error propagates automatically

---

### Step 5: Cancellation Behavior (Huge Difference)

### DispatchGroup

* No cancellation
* Work continues even if view disappears

### TaskGroup

```swift
Task {
    let data = await withTaskGroup { ... }
}
```

If parent task is cancelled:

* All child tasks are cancelled
* Resources freed immediately

Rule:

> **TaskGroup respects view & task lifecycle automatically.**

---

### Step 6: SwiftUI Example (Real World)

### DispatchGroup Anti-Pattern

```swift
.onAppear {
    let group = DispatchGroup()
    group.enter()
    loadA { group.leave() }
    group.enter()
    loadB { group.leave() }

    group.notify(queue: .main) {
        state = "Done"
    }
}
```

---

### Correct SwiftUI Pattern

```swift
.task {
    async let a = loadA()
    async let b = loadB()

    await (a, b)
    state = "Done"
}
```

SwiftUI:

* Cancels automatically
* No leaks
* MainActor-safe

---

### Step 7: Performance Considerations

* TaskGroup is lightweight
* Avoid thousands of tiny tasks
* Batch work if needed
* Avoid blocking inside tasks
* Prefer `async let` when possible

---

### Step 8: Common Migration Mistakes

* Using TaskGroup when `async let` suffices
* Updating shared state inside tasks
* Forgetting error handling
* Blocking tasks with sync I/O
* Nesting TaskGroups unnecessarily

---

### Step 9: Debugging TaskGroup Issues

Use:

* Thread Sanitizer
* Instruments → Time Profiler
* Swift Concurrency runtime warnings

Watch for:

* Long-running child tasks
* MainActor blocking
* Unhandled errors

---

### Interview Cross-Questions (Very Important)

**Q: When do you choose async let vs TaskGroup?**
A: `async let` for fixed tasks, TaskGroup for dynamic tasks.

**Q: What happens when a TaskGroup throws?**
A: Remaining child tasks are cancelled.

**Q: Does TaskGroup create threads?**
A: No — it schedules tasks on executors.

---

### One-Line Senior Summary

> “DispatchGroup counts completions; TaskGroup owns concurrent work.”

---

### Junior → Senior Mental Shift

Junior thinks:

> “Replace enter/leave with addTask.”

Senior thinks:

> “What is the scope, cancellation boundary, and error model of this concurrent work?”

---

