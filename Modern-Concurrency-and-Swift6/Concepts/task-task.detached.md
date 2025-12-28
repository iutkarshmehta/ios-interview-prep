
---

## Big Picture (One Line)

> **`Task {}` is structured and inherits context.
> `Task.detached {}` is unstructured and isolated.**

Everything else flows from this.

---

## 1. `Task {}` — Structured Task

### What It Is

```swift
Task {
    await doWork()
}
```

This creates a **child task** of the current context.

It is part of **structured concurrency**.

---

### What `Task {}` Inherits

A `Task {}` inherits from its parent:

* Cancellation
* Priority
* Actor / executor context
* Task-local values

This inheritance is intentional and critical for correctness.

---

### Lifecycle Behavior

If the parent task is:

* Cancelled → child task is cancelled
* Finished → child must finish before scope ends

This ensures:

* No leaked async work
* Predictable lifetime

---

### Actor Context Example

```swift
@MainActor
func load() {
    Task {
        print(Thread.current) // Main thread
    }
}
```

The task runs on the **MainActor** automatically.

---

### When to Use `Task {}`

Use it when:

* You want structured concurrency
* Work is tied to current scope
* You want automatic cancellation
* You want actor isolation to apply

This should be your **default choice**.

---

## 2. `Task.detached {}` — Unstructured Task

### What It Is

```swift
Task.detached {
    await doWork()
}
```

This creates a **completely independent task**.

It has **no parent**.

---

### What `Task.detached {}` Does NOT Inherit

Detached tasks do **not** inherit:

* Cancellation
* Priority
* Actor context
* Task-local values

They start in a **clean, isolated environment**.

---

### Lifecycle Behavior

* Runs until completion or manual cancellation
* Parent finishing does nothing
* Parent cancellation does nothing

You must manage:

* Cancellation
* Lifetime
* Side effects

---

### Actor Context Example

```swift
@MainActor
func load() {
    Task.detached {
        print(Thread.current) // NOT guaranteed main thread
    }
}
```

Actor isolation is **broken intentionally**.

---

## Why `Task.detached` Is Dangerous

Detached tasks:

* Can outlive UI
* Can outlive features
* Can update invalid state
* Can leak work

They bring back **fire-and-forget concurrency**.

---

## Cancellation Comparison

### `Task {}`

```swift
Task {
    await longTask()
}
```

* Cancel parent → task cancels
* `Task.isCancelled` works
* Cooperative cancellation flows

---

### `Task.detached {}`

```swift
Task.detached {
    await longTask()
}
```

* Cancellation does NOT propagate
* Must cancel manually
* Easy to forget

---

## Priority Behavior

### `Task {}`

* Inherits parent priority
* Priority adjustments propagate

### `Task.detached {}`

* Uses default priority
* Must set explicitly

```swift
Task.detached(priority: .background) {
    await doWork()
}
```

---

## Structured vs Unstructured (Core Difference)

| Aspect                   | Task {} | Task.detached {} |
| ------------------------ | ------- | ---------------- |
| Parent task              | Yes     | No               |
| Structured               | Yes     | No               |
| Cancellation inheritance | Yes     | No               |
| Actor inheritance        | Yes     | No               |
| Safe default             | Yes     | No               |

---

## Real-World Example (SwiftUI)

### Correct

```swift
.task {
    await loadData()
}
```

* Cancelled when view disappears
* Safe

---

### Dangerous

```swift
.task {
    Task.detached {
        await loadData()
    }
}
```

* Detached from view lifecycle
* Cancels nothing
* Bug-prone

---

## When to Use `Task.detached` (Rare)

Use only when:

* Work must survive caller lifetime
* Background system tasks
* Analytics uploads
* Logging
* Fire-and-forget operations

Even then, document it clearly.

---

## Staff Engineer Rule of Thumb

> **If you are unsure, do NOT use `Task.detached`.**

---

## Mental Model

* `Task {}` → child task, scoped, safe
* `Task.detached {}` → free-floating, unmanaged

---

## Final Takeaway

Swift Concurrency is designed around **structure**.

`Task.detached` breaks that structure deliberately — which is why it is powerful but dangerous.

---


