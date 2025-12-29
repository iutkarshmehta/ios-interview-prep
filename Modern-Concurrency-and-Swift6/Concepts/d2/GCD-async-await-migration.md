### GCD → async/await Migration (Swift Concurrency)
#### Why Migration (First Principles)

GCD solved **threading**, not **concurrency correctness**.

Problems with GCD:

* No ownership model
* Easy to cause race conditions
* Manual thread hopping
* Cancellation is hard
* Callback hell

Swift Concurrency solves:

* **Correctness first**
* Structured lifetimes
* Automatic cancellation
* Compiler-enforced safety

Goal of migration:

> **Move from “manage threads” → “describe async work safely”**

---

### Mental Model Shift (Very Important)

| GCD                  | async/await              |
| -------------------- | ------------------------ |
| Threads              | Tasks                    |
| Queues               | Executors                |
| Callbacks            | Suspension points        |
| Manual cancellation  | Cooperative cancellation |
| Developer discipline | Compiler enforcement     |

Key mindset:

> **You stop thinking about queues and start thinking about ownership and suspension.**

---

### Step-by-Step Migration Strategy (Follow This Always)

1. Identify async boundaries
2. Convert completion handlers → `async`
3. Replace `DispatchQueue.main.async`
4. Remove explicit queues
5. Introduce actors where needed
6. Verify cancellation behavior
7. Enable strict concurrency & TSAN

---

### Step 1: Identify GCD Patterns to Migrate

Common patterns:

* `DispatchQueue.global().async`
* `DispatchQueue.main.async`
* `DispatchGroup`
* Semaphores
* Nested callbacks

---

### Step 2: Completion Handler → async/await

### GCD Version

```swift
func fetchUser(completion: @escaping (User?, Error?) -> Void) {
    DispatchQueue.global().async {
        let user = User(name: "Utkarsh")
        DispatchQueue.main.async {
            completion(user, nil)
        }
    }
}
```

Problems:

* Thread hopping
* Completion coupling
* Hard to cancel

---

### Migrated Version (async/await)

```swift
func fetchUser() async throws -> User {
    User(name: "Utkarsh")
}
```

Usage:

```swift
Task {
    let user = try await fetchUser()
    updateUI(user)
}
```

Benefits:

* Linear code
* Cancellation-aware
* Compiler-checked

---

### Step 3: Replacing `DispatchQueue.main.async`

### GCD

```swift
DispatchQueue.main.async {
    self.title = "Hello"
}
```

### Swift Concurrency

```swift
await MainActor.run {
    self.title = "Hello"
}
```

Or better:

```swift
@MainActor
func updateTitle() {
    title = "Hello"
}
```

Rule:

> **UI updates must live on MainActor, not sprinkled across code.**

---

### Step 4: Background Work Without Queues

### GCD

```swift
DispatchQueue.global(qos: .userInitiated).async {
    heavyWork()
}
```

### async/await

```swift
Task(priority: .userInitiated) {
    await heavyWork()
}
```

Difference:

* Task is cancellable
* Priority propagates
* Structured lifetime

---

### Step 5: DispatchGroup → async let / TaskGroup

### GCD Version

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

---

### async let Version (Preferred)

```swift
async let a = fetchA()
async let b = fetchB()

let (resultA, resultB) = await (a, b)
updateUI()
```

---

### TaskGroup Version (Dynamic Tasks)

```swift
await withTaskGroup(of: Data.self) { group in
    for url in urls {
        group.addTask {
            await fetch(url)
        }
    }
}
```

---

### Step 6: Semaphores → Actors

### GCD Semaphore (Dangerous)

```swift
let semaphore = DispatchSemaphore(value: 1)

semaphore.wait()
sharedData += 1
semaphore.signal()
```

Problems:

* Deadlocks
* Priority inversion
* Hard to debug

---

### Actor Replacement

```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

Benefits:

* No deadlocks
* Compiler-enforced safety
* Clear ownership

---

### Step 7: Cancellation Migration

### GCD (Manual, Often Ignored)

```swift
DispatchWorkItem {
    if workItem.isCancelled { return }
}
```

---

### async/await (Built-in)

```swift
Task {
    try await Task.sleep(for: .seconds(5))
}
```

If cancelled:

* Throws `CancellationError`
* Stops automatically

Rule:

> **Cancellation is free if you `await`.**

---

### Step 8: Migrating Legacy APIs Using Continuations

### Old API

```swift
func loadData(completion: @escaping (Data?) -> Void)
```

---

### `withCheckedContinuation`

```swift
func loadData() async -> Data {
    await withCheckedContinuation { continuation in
        loadData { data in
            continuation.resume(returning: data!)
        }
    }
}
```

Compiler ensures:

* Resume exactly once
* No leaks

---

### Step 9: SwiftUI Migration Pattern

### GCD Anti-Pattern

```swift
.onAppear {
    DispatchQueue.global().async {
        let data = load()
        DispatchQueue.main.async {
            self.state = data
        }
    }
}
```

---

### Correct SwiftUI Pattern

```swift
.task {
    let data = await load()
    state = data
}
```

Automatic:

* Lifecycle handling
* Cancellation
* MainActor safety

---

### Common Migration Mistakes (Very Important)

* Mixing GCD and async/await
* Leaving `DispatchQueue.main.async` everywhere
* Using `Task.detached` unnecessarily
* Ignoring cancellation
* Migrating syntax without changing design

---

### Performance Considerations During Migration

* Avoid over-creating tasks
* Avoid MainActor blocking
* Batch UI updates
* Prefer actors over locks
* Measure with Instruments

---

### Debugging During Migration

Tools to enable:

* Thread Sanitizer
* Main Thread Checker
* Swift Concurrency logs

Look for:

* Accidental MainActor work
* Actor contention
* Priority inversion

---

### Interview Cross-Questions (Prepare These)

**Q: Why is async/await safer than GCD?**
A: Structured concurrency, cancellation propagation, and compiler-enforced safety.

**Q: Can async/await still have race conditions?**
A: Yes, if ownership is wrong or actors are misused.

**Q: Do async functions create threads?**
A: No, they suspend and resume on executors.

---

### One-Line Senior Summary

> “Migrating from GCD to async/await is not syntax replacement — it’s a shift to structured, cancellable, compiler-verified concurrency.”

---

### Junior → Senior Insight

Junior thinks:

> “Replace queues with async.”

Senior thinks:

> “Who owns this state, what cancels it, and what blocks the MainActor?”

---

### What You’re Ready For Next

* Mixed GCD + async debugging
* Actor reentrancy pitfalls
* Large-scale migration strategy
* Testing async code
* Performance tuning post-migration

