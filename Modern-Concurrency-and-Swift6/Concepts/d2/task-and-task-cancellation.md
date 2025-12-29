### SwiftUI `.task`

Think of `.task` as **SwiftUI’s structured concurrency entry point**.
It attaches an **async task to a view’s lifecycle**, not to a thread and not to an object.

Core rule:

> **If the view exists, the task exists.
> If the view disappears, the task is cancelled.**

Basic example:

```swift
struct ProfileView: View {
    @State private var name = ""

    var body: some View {
        Text(name)
            .task {
                name = await loadProfileName()
            }
    }
}
```

What SwiftUI does:

* Creates a `Task`
* Runs it when the view appears
* Cancels it when the view is removed from the hierarchy

No manual cleanup required.

---

### `.task` Is Structured Concurrency

`.task` creates a **child task** of SwiftUI’s internal task tree.

Properties:

* Inherits priority
* Inherits cancellation
* Automatically cleaned up

This is **not** the same as `Task.detached`.

---

### When `.task` Runs (Important Detail)

`.task` runs:

* When the view first appears
* When the view is **re-created**

SwiftUI is value-based.
Views can be destroyed and recreated frequently.

Example:

```swift
if isLoggedIn {
    DashboardView()
        .task {
            await fetchData()
        }
}
```

Toggling `isLoggedIn`:

* Cancels old task
* Starts a new one

---

### `.task(id:)` — Controlled Re-Execution

```swift
.task(id: userID) {
    await loadUser(id: userID)
}
```

Behavior:

* Task runs when view appears
* Task re-runs **only when `userID` changes**
* Previous task is cancelled automatically

This is the preferred pattern for:

* Network requests
* Search
* Pagination

---

### `.task` Cancellation Behavior

When a view:

* Disappears
* Is replaced
* Changes identity

SwiftUI sends a **cancellation signal** to the task.

The task is not force-killed.
It is **politely asked to stop**.

---

### How Cancellation Works in Practice

Cancellation is **cooperative**.

```swift
.task {
    try await Task.sleep(for: .seconds(5))
    print("Finished")
}
```

If the view disappears before 5 seconds:

* `Task.sleep` throws `CancellationError`
* Task exits automatically

---

### Cancellation-Aware APIs (Auto-Cancel)

These APIs automatically stop when cancelled:

* `Task.sleep`
* `URLSession` async APIs
* `AsyncSequence` iteration
* `await` actor calls

Example:

```swift
.task {
    let data = try await URLSession.shared.data(from: url)
    // Cancelled? This never completes
}
```

---

### Manual Cancellation Checks (Critical)

For long-running CPU work:

```swift
.task {
    for i in 0..<1_000_000 {
        if Task.isCancelled {
            return
        }
        heavyWork(i)
    }
}
```

Rule:

> If your task does not `await`, you **must** check for cancellation.

---

### `defer` Runs on Cancellation

```swift
.task {
    defer {
        print("Cleanup runs")
    }

    try await Task.sleep(for: .seconds(10))
}
```

When cancelled:

* `CancellationError` is thrown
* `defer` executes
* Resources are released

This is essential for:

* Closing streams
* Invalidating timers
* Releasing locks

---

### `.task` vs `.onAppear`

```swift
.onAppear {
    Task {
        await load()
    }
}
```

Problems:

* Task is not tied to view lifecycle
* Manual cancellation required
* Easy to leak work

`.task`:

* Lifecycle-aware
* Automatically cancelled
* Structured and safe

---

### `.task` + `AsyncSequence`

```swift
.task {
    for await event in eventStream {
        handle(event)
    }
}
```

Behavior:

* View disappears → task cancelled
* Stream terminates
* `onTermination` fires

This is the **correct** way to consume streams in SwiftUI.

---

### Common Bug Example (Interview Favorite)

```swift
.task {
    while true {
        await pollServer()
    }
}
```

Bug:

* Infinite loop
* No cancellation check

Correct:

```swift
.task {
    while !Task.isCancelled {
        await pollServer()
    }
}
```

---

### `.task` and `@MainActor`

`.task` runs on an **unspecified executor**.

```swift
.task {
    title = "Hello" // ❌ MainActor violation
}
```

Correct:

```swift
.task {
    await MainActor.run {
        title = "Hello"
    }
}
```

Or:

```swift
.task {
    await viewModel.load()
}
```

With:

```swift
@MainActor
func load() { }
```

---

### Senior Engineer Mental Model

* `.task` is **owned by the view**
* Cancellation is **automatic and cooperative**
* `await` points are **cancellation points**
* CPU loops must check `Task.isCancelled`
* Prefer `.task(id:)` for predictable behavior

---

### When to Use `.task`

* Initial data loading
* Streaming updates
* Responding to state changes
* Async side effects tied to UI

---

