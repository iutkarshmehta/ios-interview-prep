

### First: What Is a Continuation?

A **continuation** represents:

> **“The rest of an async function from a suspension point onward.”**

In simple terms:

* When an async function suspends, Swift saves:

  * Local variables
  * Where execution should resume
* That saved state is the **continuation**

A continuation lets *someone else* resume that suspended function later.

---

### Why Continuations Exist

Swift did **not rewrite the world**.

Most APIs are still:

* Callback-based
* Delegate-based
* Completion-handler-based

Example:

```swift
func fetchUser(completion: @escaping (User?, Error?) -> Void)
```

But modern Swift wants:

```swift
func fetchUser() async throws -> User
```

**Continuations are the bridge.**

---

### Key Rule (Very Important)

> **A continuation must be resumed exactly once.**

* Resume twice → crash
* Never resume → memory leak / deadlock

This is why “checked” continuations exist.

---

### 1. `withCheckedContinuation`

### What It Is

Used when:

* The callback returns a value
* No error is involved

Signature (conceptual):

```swift
withCheckedContinuation { continuation in
    // resume later
}
```

---

### Example: Callback → async (No Errors)

Old API:

```swift
func fetchNumber(completion: @escaping (Int) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        completion(42)
    }
}
```

Modern async version:

```swift
func fetchNumber() async -> Int {
    await withCheckedContinuation { continuation in
        fetchNumber { value in
            continuation.resume(returning: value)
        }
    }
}
```

### What Happens Internally

1. `await` suspends the function
2. Swift hands you a continuation
3. Callback fires later
4. You resume the continuation
5. Async function continues

---

### 2. `withCheckedThrowingContinuation`

### What It Is

Used when:

* The callback can succeed or fail
* Errors must be propagated

Signature (conceptual):

```swift
withCheckedThrowingContinuation { continuation in
    // resume or throw
}
```

---

### Example: Callback → async (With Errors)

Old API:

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

Async version:

```swift
func fetchUser() async throws -> User {
    try await withCheckedThrowingContinuation { continuation in
        fetchUser { user, error in
            if let user {
                continuation.resume(returning: user)
            } else {
                continuation.resume(throwing: error!)
            }
        }
    }
}
```

---

### Why “Checked” Matters

Swift provides:

* `withCheckedContinuation`
* `withCheckedThrowingContinuation`

They:

* Detect double resume
* Warn if never resumed
* Crash in debug to surface bugs early

Unchecked continuations exist, but **should almost never be used**.

---

### Common Mistakes (Very Important)

### ❌ Resuming Twice

```swift
continuation.resume(returning: value)
continuation.resume(returning: value) // ❌ crash
```

---

### ❌ Not Resuming on All Paths

```swift
if success {
    continuation.resume(returning: value)
}
// else → continuation never resumed ❌
```

---

### ❌ Storing Continuation Incorrectly

Never store continuations beyond their intended scope unless you **fully control lifecycle**.

---

### Continuations Do NOT Create Concurrency

Important clarification:

* Continuations do not create threads
* They do not run work in parallel
* They only bridge execution models

---

### Cancellation and Continuations

Continuations **do not auto-cancel**.

If the task is cancelled:

* Your callback may still fire
* You must check `Task.isCancelled` manually

Example:

```swift
if Task.isCancelled {
    continuation.resume(throwing: CancellationError())
    return
}
```

---

### When to Use Continuations

Use continuations when:

* Wrapping legacy APIs
* Migrating code incrementally
* Interfacing with C / Objective-C APIs
* SDKs without async/await support

Do **not** use them for new APIs.

---

### Mental Model to Remember

> **A continuation is a handle to resume a suspended async function later.**

---

### Staff-Level Insight

Continuations are **powerful and dangerous**:

* Powerful because they let you integrate anything
* Dangerous because Swift cannot enforce correctness beyond “resume once”

That’s why Swift makes you opt-in explicitly.

---


