

---

# Swift Concurrency Interview Notes

# Structured vs Unstructured Concurrency

## Structured Concurrency

Structured concurrency means:

> Child tasks are **bound to the scope of the parent task**, and the parent **cannot exit until all child tasks complete or are cancelled**.

### Key Properties

1. **Parent–child relationship**

```text
Parent Task
   ├── Child Task
   ├── Child Task
   └── Child Task
```

2. **Child tasks cannot outlive the parent scope**

Swift guarantees:

```
Parent cannot exit
until child tasks finish
```

3. **Cancellation propagates**

```
Parent cancelled
      ↓
Child tasks cancelled
```

---

## APIs that use Structured Concurrency

### 1. `async let`

```swift
func load() async {
    async let user = fetchUser()
    async let posts = fetchPosts()

    let result = await (user, posts)
}
```

Properties:

* Runs tasks **concurrently**
* Bound to **function scope**
* Parent **waits before exiting**

---

### 2. `TaskGroup`

```swift
await withTaskGroup(of: Int.self) { group in
    group.addTask { await fetchA() }
    group.addTask { await fetchB() }
}
```

Properties:

* Multiple **child tasks**
* Parent **waits until all tasks complete**

---

# Unstructured Concurrency

Tasks are **not bound to the parent scope**.

Parent does **not wait** for them.

---

## 1. `Task {}`

```swift
func load() {
    Task {
        await fetchUser()
    }

    print("Function finished")
}
```

Possible execution:

```
Function finished
fetchUser completes later
```

### Characteristics

* Inherits **actor context**
* Inherits **priority**
* Does **NOT enforce structured lifetime**

---

## 2. `Task.detached`

```swift
Task.detached {
    await fetchUser()
}
```

### Characteristics

* Does **not inherit actor context**
* Does **not inherit priority**
* Completely **independent task**

---

# Summary Table

| API             | Structured? | Actor Context | Parent Waits |
| --------------- | ----------- | ------------- | ------------ |
| `async let`     | ✅ Yes       | Yes           | Yes          |
| `TaskGroup`     | ✅ Yes       | Yes           | Yes          |
| `Task {}`       | ❌ No        | Yes           | No           |
| `Task.detached` | ❌ No        | No            | No           |

---

# Interview One-Line Summary

> Structured concurrency ensures tasks are tied to the scope that created them, while unstructured tasks (`Task` and `Task.detached`) can outlive their parent scope.

---


