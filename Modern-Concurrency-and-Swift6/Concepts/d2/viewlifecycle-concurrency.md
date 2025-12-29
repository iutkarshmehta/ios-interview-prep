### SwiftUI View Lifecycle & Concurrency

---

### Why This Topic Exists (First Principles)

SwiftUI is:

* **Value-type UI**
* **Declarative**
* **State-driven**

Concurrency exists because:

* UI must stay responsive
* Data arrives asynchronously
* Work must be cancellable and safe

The problem:

> SwiftUI **recreates views frequently**, while async work may still be running.

So SwiftUI needed:

* Lifecycle-aware async execution
* Automatic cancellation
* Deterministic behavior

That’s where **`.task` + structured concurrency** fit in.

---

### What a SwiftUI View *Really* Is

```swift
struct ProfileView: View {
    var body: some View {
        Text("Hello")
    }
}
```

This is **not** a screen.
This is **not** a controller.
This is a **temporary description** of UI.

Important rule:

> SwiftUI views are **cheap, transient, and recreated often**.

---

### View Identity (Critical for Concurrency)

SwiftUI decides view identity using:

* Type
* Position in the hierarchy
* Explicit `id`

Example:

```swift
if isLoggedIn {
    DashboardView()
} else {
    LoginView()
}
```

Toggling `isLoggedIn`:

* Destroys one view
* Creates another
* Cancels attached tasks

---

### SwiftUI View Lifecycle (Reality, Not UIKit)

SwiftUI does **not** have:

* `viewDidLoad`
* `viewWillAppear`
* `viewDidDisappear`

Instead:

| Event           | Meaning                         |
| --------------- | ------------------------------- |
| View appears    | SwiftUI inserts it in hierarchy |
| Body recomputed | State changed                   |
| View disappears | Removed from hierarchy          |
| View destroyed  | Identity lost                   |

Concurrency hooks into **these transitions**.

---

### `.task` and View Lifecycle

```swift
Text("Profile")
    .task {
        await loadProfile()
    }
```

What happens:

1. View appears
2. SwiftUI creates a **child task**
3. Task runs concurrently
4. View disappears → task is cancelled

You do **not** manage this manually.

---

### `.task(id:)` — Lifecycle + State Driven

```swift
.task(id: userID) {
    await loadUser(id: userID)
}
```

Behavior:

* Runs on appear
* Re-runs only when `userID` changes
* Cancels previous task automatically

This solves:

* Multiple fetches
* Stale data
* Manual cancellation bugs

---

### Why `.onAppear` Is Dangerous with Concurrency

```swift
.onAppear {
    Task {
        await loadData()
    }
}
```

Problem:

* Task is **detached from view lifecycle**
* View can disappear
* Task keeps running
* Memory + logic leaks

Correct:

```swift
.task {
    await loadData()
}
```

---

### Cancellation: The Heart of View Concurrency

When a view disappears:

* SwiftUI sends **cancellation signal**
* Task is not killed
* Task must **cooperate**

---

### Cancellation Happens at `await` Points

```swift
.task {
    try await Task.sleep(for: .seconds(10))
    print("Done")
}
```

If view disappears:

* `Task.sleep` throws `CancellationError`
* Code after it never runs

This is safe by default.

---

### CPU Work Requires Manual Checks

```swift
.task {
    for i in 0..<1_000_000 {
        if Task.isCancelled {
            return
        }
        heavyCompute(i)
    }
}
```

Rule:

> If you don’t `await`, you must check cancellation yourself.

---

### `defer` Always Runs (Even on Cancellation)

```swift
.task {
    defer {
        print("Cleanup")
    }

    try await Task.sleep(for: .seconds(5))
}
```

Guarantees:

* Resource cleanup
* Stream termination
* Timer invalidation

---

### View Updates Must Be `@MainActor`

```swift
.task {
    title = "Hello" // ❌
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

Better:

```swift
@MainActor
func load() {
    title = "Hello"
}
```

And:

```swift
.task {
    await viewModel.load()
}
```

---

### ViewModel Concurrency Ownership

Correct pattern:

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var name = ""

    func load() async {
        name = "Utkarsh"
    }
}
```

Why:

* ViewModel owns UI state
* Actor isolation prevents races
* View remains dumb

---

### Common Junior Mistakes (Very Important)

❌ Doing networking directly in the view
❌ Using `Task.detached` in SwiftUI
❌ Forgetting cancellation
❌ Mutating UI state off MainActor
❌ Assuming view is long-lived

---

### Mental Model (Memorize This)

* Views are **descriptions**
* State drives UI
* `.task` is lifecycle-aware
* Cancellation is automatic
* `await` = cancellation point
* MainActor protects UI

---

### Interview-Grade Explanation (Say This)

> “SwiftUI views are value types and recreated frequently.
> `.task` ties async work to the view’s lifecycle using structured concurrency.
> When a view disappears, SwiftUI cancels the task cooperatively, preventing leaks and stale updates.”

---

### Senior Engineer Rulebook

* One source of truth
* One owner of state
* No detached tasks in views
* Prefer `.task(id:)`
* ViewModel owns concurrency logic

---
