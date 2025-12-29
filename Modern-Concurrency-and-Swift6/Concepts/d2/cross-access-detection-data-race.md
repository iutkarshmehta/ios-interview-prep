### Cross-Actor Data Access

An **actor** is a concurrency primitive that **owns mutable state** and guarantees **serialized access** to it.

```swift
actor BankAccount {
    private var balance: Int = 0

    func deposit(_ amount: Int) {
        balance += amount
    }

    func currentBalance() -> Int {
        balance
    }
}
```

Here, `balance` is **actor-isolated**.

---

### Illegal Cross-Actor Access (Compile-Time Error)

```swift
let account = BankAccount()

print(account.balance) 
// ❌ Error: Actor-isolated property 'balance' cannot be accessed
```

Why this is rejected:

* You are trying to synchronously read data
* The actor may be executing another task
* Swift refuses to guess about safety

---

### Correct Cross-Actor Access (Using `await`)

```swift
let account = BankAccount()

Task {
    await account.deposit(100)
    let value = await account.currentBalance()
    print(value)
}
```

What `await` means:

* The call may suspend
* The actor processes one message at a time
* State remains consistent

---

### Why `await` Is Required Even for Reads

```swift
func currentBalance() -> Int {
    balance
}
```

Even reads:

* Must wait for prior writes to finish
* Must respect actor serialization

Consistency > convenience.

---

### `nonisolated` Example

```swift
actor APIClient {
    let baseURL = "https://example.com"

    nonisolated func endpoint(path: String) -> String {
        baseURL + path
    }
}
```

Why this works:

* No mutable state is accessed
* Safe to call synchronously
* No actor hop required

---

### `@MainActor` as a Special Actor (UI)

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var name: String = ""
}
```

```swift
Task {
    // ❌ compile-time error
    viewModel.name = "Utkarsh"
}
```

```swift
Task {
    await MainActor.run {
        viewModel.name = "Utkarsh"
    }
}
```

UI state is protected the same way as any actor.

---

### Data Race Detection

A **data race** happens when multiple tasks access shared mutable memory without coordination.

---

### Classic Race Condition (Pre-Concurrency)

```swift
var counter = 0

Task.detached {
    counter += 1
}

Task.detached {
    counter += 1
}
```

Problem:

* Two tasks mutate the same variable
* Execution order is undefined
* Result is unpredictable

---

### Swift Rejects This in Strict Concurrency

```swift
var counter = 0

Task {
    counter += 1
}
// ❌ Error: Mutation of captured var 'counter' in concurrently-executing code
```

The compiler stops you **before the bug exists**.

---

### Fix Using an Actor

```swift
actor SafeCounter {
    private var value = 0

    func increment() {
        value += 1
    }

    func get() -> Int {
        value
    }
}
```

```swift
let counter = SafeCounter()

Task {
    await counter.increment()
}
```

Serialized access → zero races.

---

### `Sendable` and Data Safety

When data crosses concurrency boundaries, it must be **safe to copy or share**.

```swift
struct User: Sendable {
    let id: Int
    let name: String
}
```

Why safe:

* Value type
* Immutable properties

---

### Non-Sendable Example (Rejected)

```swift
final class Session {
    var token: String
    init(token: String) {
        self.token = token
    }
}
```

```swift
Task {
    use(session)
}
// ❌ Error: Type 'Session' is not Sendable
```

---

### `@unchecked Sendable` (Expert-Level Escape Hatch)

```swift
final class ThreadSafeCache: @unchecked Sendable {
    private let lock = NSLock()
    private var store: [String: Int] = [:]
}
```

You promise:

* Internal synchronization is correct
* The compiler trusts you
* Bugs here are **your responsibility**

---

### How Cross-Actor Access Enables Data Race Detection

Actors define **who owns the data**
`await` defines **when access is allowed**
`Sendable` defines **what can cross boundaries**

Together they create:

* Compile-time race prevention
* Deterministic behavior
* Scalable concurrency

---

### Senior Engineer Mental Model

> “Every piece of mutable state must have exactly one owner.
> Actors enforce ownership.
> The compiler enforces discipline.”

---

