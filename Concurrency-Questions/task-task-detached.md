
---

# Swift Concurrency Interview Notes

## Question 3: `Task {}` vs `Task.detached {}`

### Your Key Idea (Mostly Correct)

You said:

* `Task {}` inherits actor context ✅
* `Task.detached` does not inherit actor context ✅
* Cancellation propagation differs ✅
* Prefer `Task` by default ✅

Let’s refine each part.

---

# 1. Actor Context Inheritance

### `Task {}`

```swift
Task {
    await loadData()
}
```

* Inherits **actor context**
* Inherits **priority**
* Inherits **task-local values**

Example:

```swift
@MainActor
func load() {
    Task {
        updateUI()
    }
}
```

Here:

```
Task runs on MainActor
```

because it **inherits the actor context**.

---

### `Task.detached`

```swift
Task.detached {
    await loadData()
}
```

* **Does NOT inherit actor**
* **Does NOT inherit priority**
* **Does NOT inherit task-local values**

It runs on a **separate executor**.

Example:

```swift
@MainActor
func load() {

    Task.detached {
        self.title = "Hello"
    }
}
```

This causes:

```
Actor isolation violation
```

because it **lost the MainActor context**.

---

# 2. Cancellation Propagation

### `Task {}`

Cancellation **propagates from parent task**.

```
Parent Task
   │
   └── Task {}
```

If parent is cancelled → child is cancelled.

---

### `Task.detached`

No parent relationship.

```
Parent Task

Detached Task
```

So:

* Parent cancellation **does not cancel it automatically**
* You must **manually handle cancellation**

---

# 3. Structured vs Unstructured

Small correction to your statement.

| Type            | Concurrency Type                        |
| --------------- | --------------------------------------- |
| `async let`     | Structured                              |
| `TaskGroup`     | Structured                              |
| `Task {}`       | **Unstructured but context inheriting** |
| `Task.detached` | **Fully detached unstructured**         |

So `Task {}` is **not strictly structured**, but it **inherits context**, which makes it safer.

---

# 4. When to Use

### Prefer `Task {}`

Default choice.

Example:

* UI triggers async work
* ViewModel calling APIs

```swift
Task {
    await fetchData()
}
```

---

### Use `Task.detached` rarely

When you want **completely independent work**.

Example:

* Background cleanup
* Logging
* Analytics

```swift
Task.detached {
    await processLargeFile()
}
```

---

# Interview Ready Answer

> "`Task {}` inherits actor context, priority, and cancellation from its parent task, making it safer for most use cases. `Task.detached` creates an independent task that does not inherit actor context or cancellation, so it should only be used when completely detached background work is required."

---

# Quick Interview Trick Question

Consider this:

```swift
@MainActor
class VM {

    func load() {

        Task {
            self.title = "Loaded"
        }

    }

    var title = ""
}
```

Question:

Will this cause an **actor isolation violation**?

A) Yes
B) No

What do you think?
