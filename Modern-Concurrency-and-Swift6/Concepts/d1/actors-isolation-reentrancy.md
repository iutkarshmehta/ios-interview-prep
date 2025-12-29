Perfect — this is one of the **most important parts of Swift Concurrency**.
I’ll teach this the same way I mentor engineers on my team: **from intuition → rules → sharp edges → real-world guidance**.

Take your time reading this. Actors look simple, but mastery is here.

---

### 1. Actor Basics

### The problem actors solve

The core problem in concurrent systems is **shared mutable state**.

Before actors:

* Multiple threads access the same data
* You add locks, queues, semaphores
* Code *looks correct* but breaks under load

Example (pre-actors):

```swift
class Counter {
    var value = 0
}
```

If two threads modify `value` at the same time → **data race**.

---

### What an actor is

An **actor** is like a class, but with **one critical guarantee**:

> Only **one task at a time** can access an actor’s mutable state.

```swift
actor Counter {
    var value = 0

    func increment() {
        value += 1
    }
}
```

You do **not**:

* Write locks
* Manage queues
* Think about threads

The compiler + runtime enforce safety.

---

### How actors work internally (mental model)

Think of an actor as:

* Owning its own **serial executor**
* Tasks line up to access it
* One task runs inside the actor at a time

But unlike GCD:

* You don’t control the queue
* You can’t accidentally misuse it
* Safety is enforced at compile time

---

### How you interact with actors

Accessing an actor **from outside** is always asynchronous:

```swift
let counter = Counter()
await counter.increment()
```

Why?

* The call might need to wait for exclusive access
* Swift forces you to acknowledge this with `await`

---

### 2. Actor Isolation

Actor isolation is where most confusion happens — so let’s slow down.

---

### What is actor isolation?

**Actor isolation** means:

> Actor state can only be accessed **from within the actor itself**, unless explicitly allowed.

This is enforced by the compiler.

---

### Isolated vs non-isolated code

### Inside the actor

```swift
actor BankAccount {
    var balance = 0

    func deposit(_ amount: Int) {
        balance += amount // SAFE
    }
}
```

* No `await`
* Direct access
* Guaranteed exclusive access

---

### Outside the actor

```swift
let account = BankAccount()
print(await account.balance) // async access
```

* Must use `await`
* Compiler-enforced
* Prevents races

---

### Compile-time safety example

This is **not allowed**:

```swift
func printBalance(account: BankAccount) {
    print(account.balance) // ❌ compiler error
}
```

Why?

* You’re trying to access isolated state synchronously
* Swift blocks this at compile time

This is a **huge improvement** over GCD, where this would compile and fail at runtime.

---

### `nonisolated` members

Sometimes you want logic that:

* Belongs to the actor
* Does NOT touch its state

```swift
actor Logger {
    let prefix = "LOG"

    nonisolated func format(_ message: String) -> String {
        "\(prefix): \(message)"
    }
}
```

Rules:

* `nonisolated` cannot access actor state
* Can be called synchronously
* Must be thread-safe by design

---

### `@MainActor` and isolation

`@MainActor` is a **special actor** bound to the main thread.

```swift
@MainActor
class ViewModel {
    var title = ""
}
```

Guarantee:

* All access happens on main thread
* UI safety enforced by compiler

---

### 3. Actor Reentrancy (This is the tricky part)

This is where many developers get surprised.

---

### What is reentrancy?

An actor is **reentrant**.

That means:

> While an actor is awaiting, **another task may enter the actor**.

This is intentional.

---

### Example of reentrancy

```swift
actor UserStore {
    var users: [String] = []

    func addUser() async {
        users.append("A")
        await fetchRemoteData()
        users.append("B")
    }
}
```

Timeline:

1. Task 1 enters `addUser`
2. Appends `"A"`
3. Hits `await` → actor is suspended
4. Task 2 enters actor
5. Modifies `users`
6. Task 1 resumes

State may have changed.

---

### Why Swift allows this

If actors were **non-reentrant**:

* One slow `await` would block all access
* Deadlocks become common
* Performance collapses

Reentrancy is a **performance tradeoff** for safety.

---

### The rule you MUST remember

> **Actor state is only guaranteed to be consistent between `await` points.**

Never assume state is unchanged across an `await`.

---

### How to write safe actor code

### Bad pattern

```swift
func transfer() async {
    if balance >= 100 {
        await withdraw(100)
    }
}
```

Between the check and withdraw:

* Another task may change `balance`

---

### Good pattern

```swift
func withdrawIfPossible(_ amount: Int) async -> Bool {
    guard balance >= amount else { return false }
    balance -= amount
    return true
}
```

* No `await` in the critical section
* State change is atomic from actor’s perspective

---

### Reentrancy-safe thinking

Inside actors:

* Keep critical sections **sync**
* Move `await` outside state mutation
* Snapshot state before awaiting if needed

---

### Staff Engineer Summary (Memorize This)

* Actors prevent data races
* Actor isolation is enforced by the compiler
* Actor access from outside is always async
* Actors are reentrant by design
* State can change across `await`
* Write actor methods to be await-safe

---

### Final Mental Model

> **Actors replace locks with rules.
> Reentrancy replaces blocking with correctness.**

If you understand actors deeply, you understand **why Swift Concurrency is safe by default**.

---

