### Completion Handlers → async/await (Swift Concurrency)
### Why Completion Handlers Had to Die (First Principles)

Completion handlers were designed when:

* Swift had no concurrency model
* GCD was the only tool
* Safety was the developer’s responsibility

Problems with completion handlers:

* Callback hell
* Inverted control flow
* Easy to forget calling completion
* No cancellation propagation
* Hard to compose multiple async operations
* Weak error handling

Core issue:

> **Completion handlers encode time as callbacks, not as values.**

async/await fixes this.

---

### Mental Model Shift (Critical)

| Completion Handler     | async/await            |
| ---------------------- | ---------------------- |
| Callback invoked later | Function suspends      |
| Caller loses control   | Caller retains control |
| Manual error passing   | Typed `throws`         |
| Manual cancellation    | Built-in               |
| Hard to chain          | Natural composition    |

Key idea:

> **async/await makes asynchronous code read like synchronous code.**

---

### Migration Strategy (Always Follow This Order)

1. Identify async boundaries
2. Convert completion signature → `async` or `async throws`
3. Bridge legacy APIs using continuations
4. Replace nested callbacks with linear flow
5. Move UI updates to `MainActor`
6. Verify cancellation behavior
7. Remove unused GCD code

---

### Step 1: Identify Completion Handler Shapes

Common forms:

```swift
(Result<T, Error>) -> Void
(T?, Error?) -> Void
(Bool) -> Void
() -> Void
```

Each maps cleanly to async.

---

### Step 2: Basic Completion → async

### Old Code

```swift
func fetchUser(completion: @escaping (User) -> Void) {
    DispatchQueue.global().async {
        let user = User(name: "Utkarsh")
        DispatchQueue.main.async {
            completion(user)
        }
    }
}
```

Problems:

* Thread hopping
* No cancellation
* No error handling

---

### Migrated async Version

```swift
func fetchUser() async -> User {
    User(name: "Utkarsh")
}
```

Usage:

```swift
Task {
    let user = await fetchUser()
    updateUI(user)
}
```

Cleaner, safer, cancellable.

---

### Step 3: Completion with Errors → async throws

### Old Code

```swift
func fetchUser(completion: @escaping (User?, Error?) -> Void) {
    DispatchQueue.global().async {
        if Bool.random() {
            completion(User(name: "Utkarsh"), nil)
        } else {
            completion(nil, NetworkError.failed)
        }
    }
}
```

Problems:

* Optional juggling
* Easy to misuse

---

### async throws Version

```swift
func fetchUser() async throws -> User {
    if Bool.random() {
        return User(name: "Utkarsh")
    } else {
        throw NetworkError.failed
    }
}
```

Usage:

```swift
Task {
    do {
        let user = try await fetchUser()
        updateUI(user)
    } catch {
        showError(error)
    }
}
```

---

### Step 4: Result Type → async throws

### Old Code

```swift
func fetchUser(completion: @escaping (Result<User, Error>) -> Void) {
    completion(.success(User(name: "Utkarsh")))
}
```

---

### async Version

```swift
func fetchUser() async throws -> User {
    User(name: "Utkarsh")
}
```

Rule:

> **`Result<T, Error>` becomes `async throws -> T`.**

---

### Step 5: Chaining Completion Handlers → Linear Flow

### Callback Hell

```swift
fetchUser { user in
    fetchPosts(user) { posts in
        fetchComments(posts) { comments in
            updateUI(comments)
        }
    }
}
```

Hard to read, hard to debug.

---

### async Composition

```swift
let user = await fetchUser()
let posts = await fetchPosts(user)
let comments = await fetchComments(posts)
updateUI(comments)
```

Key advantage:

> **Order is explicit and readable.**

---

### Step 6: Parallel Work (Was Hard, Now Easy)

### Completion Version

```swift
fetchA { a in
    fetchB { b in
        combine(a, b)
    }
}
```

---

### async let Version

```swift
async let a = fetchA()
async let b = fetchB()

let result = await combine(a, b)
```

This is structured, cancellable, and safe.

---

### Step 7: Bridging Legacy APIs (Continuations)

You don’t rewrite everything at once.

### Legacy API

```swift
func loadData(completion: @escaping (Data?, Error?) -> Void)
```

---

### withCheckedThrowingContinuation

```swift
func loadData() async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        loadData { data, error in
            if let data {
                continuation.resume(returning: data)
            } else {
                continuation.resume(throwing: error!)
            }
        }
    }
}
```

Compiler guarantees:

* Resume exactly once
* No forgotten callbacks

---

### Step 8: Cancellation Awareness (Big Win)

### Completion Handlers

* Cancellation is manual
* Usually ignored

### async/await

```swift
Task {
    try await Task.sleep(for: .seconds(5))
}
```

If cancelled:

* `CancellationError` thrown
* Task stops automatically

Rule:

> **If you `await`, you get cancellation for free.**

---

### Step 9: SwiftUI Migration Pattern

### Old SwiftUI Anti-Pattern

```swift
.onAppear {
    fetchUser { user in
        self.user = user
    }
}
```

Problems:

* No cancellation
* Runs multiple times

---

### Correct SwiftUI Pattern

```swift
.task {
    user = await fetchUser()
}
```

SwiftUI:

* Cancels on view disappearance
* Restarts on identity change

---

### Step 10: MainActor Enforcement

Never update UI inside completion callbacks randomly.

### Correct Pattern

```swift
@MainActor
func updateUI(_ user: User) {
    self.user = user
}
```

Then call from anywhere:

```swift
await updateUI(user)
```

---

### Common Migration Mistakes (Important)

* Leaving completion-based APIs mixed everywhere
* Wrapping async inside `DispatchQueue`
* Using `Task.detached` incorrectly
* Ignoring cancellation
* Returning `Void` where value is expected

---

### Performance Considerations

* async/await is cheaper than dispatch queues
* Avoid creating tasks per keystroke
* Prefer structured tasks
* Avoid blocking inside async functions

---

### Debugging After Migration

Use:

* Thread Sanitizer
* Swift Concurrency runtime warnings
* Instruments → Time Profiler

Watch for:

* MainActor blocking
* Actor contention
* Uncancelled tasks

---

### Interview Cross-Questions

**Q: Why is async/await safer than completion handlers?**
A: Compiler-enforced lifetimes, cancellation propagation, structured concurrency.

**Q: Do async functions run on background threads?**
A: Not necessarily — they suspend and resume on executors.

**Q: Can async/await still cause memory leaks?**
A: Yes, if tasks capture self strongly and aren’t cancelled.

---

### One-Line Senior Summary

> “Completion handlers encode time implicitly; async/await makes time explicit, structured, and safe.”

---

### Junior → Senior Mental Shift

Junior thinks:

> “Convert completion to async.”

Senior thinks:

> “What owns this work, what cancels it, and who updates the UI?”

---
