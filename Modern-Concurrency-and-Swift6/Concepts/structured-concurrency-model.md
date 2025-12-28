
# Swift Structured Concurrency Model

## What Is Structured Concurrency?

**Structured concurrency** is a concurrency model where:
- Concurrent tasks have a **well-defined hierarchy**
- Child tasks are **scoped to the lifetime of their parent**
- Async work **cannot outlive** the scope that created it

It applies the same rules of structure and lifetime that exist in synchronous code to asynchronous code.

---

## The Core Problem It Solves

Before structured concurrency:
- Async work could be started anywhere
- Tasks could outlive views, objects, or features
- Cancellation was manual and unreliable
- Reasoning about async lifetime was difficult

The root issue was **unbounded lifetime of async work**.

---

## Meaning of “Structured”

In synchronous code:

```swift
func foo() {
    bar()
}
````

Guarantees:

* `bar()` starts inside `foo()`
* `bar()` completes before `foo()` returns

Structured concurrency enforces **the same guarantees for async work**.

> Async tasks must start and finish within a known scope.

---

## Formal Definition

> **Structured concurrency is a model in which concurrent tasks form a hierarchy, and the lifetime of child tasks is bounded by the lifetime of their parent scope.**

---

## Task Hierarchy (Task Tree)

* Tasks form a **tree structure**
* Every task (except detached tasks) has a parent
* Parents are responsible for the lifetime of their children

This hierarchy enables:

* Predictable lifetimes
* Automatic cancellation
* Structured error propagation

---

## Basic Example

```swift
func loadData() async {
    let user = await fetchUser()
    let posts = await fetchPosts()
}
```

Guarantees:

* `fetchUser()` completes before moving forward
* `fetchPosts()` completes before `loadData()` returns
* No async work escapes the function

---

## Parallel Work Within a Scope

```swift
func loadData() async {
    async let user = fetchUser()
    async let posts = fetchPosts()

    let result = await (user, posts)
}
```

Structured guarantees:

* Both tasks start within `loadData`
* Both must complete before exiting the function
* Cancellation of `loadData` cancels both tasks

---

## Task Groups Are Structured

```swift
await withTaskGroup(of: Data.self) { group in
    group.addTask { fetchA() }
    group.addTask { fetchB() }
}
```

Rules:

* All child tasks must finish before the group exits
* Errors propagate to the parent
* Cancellation cancels remaining child tasks

---

## Scope-Based Lifetime Enforcement

When a structured concurrency scope ends:

* All child tasks must be completed or cancelled
* The scope cannot exit early
* The runtime enforces this rule

This prevents background work from leaking.

---

## Cancellation in Structured Concurrency

* Cancellation flows **from parent to child**
* Child tasks automatically observe cancellation
* No manual cancellation wiring required

Cancellation is reliable **because tasks are structured**.

---

## Error Handling Is Structured

* Errors remain within the scope
* A failing child can cancel sibling tasks
* Errors propagate naturally to the parent

This avoids partial-failure bugs common in callback-based systems.

---

## What Structured Concurrency Is NOT

Structured concurrency is **not**:

* About maximizing parallelism
* About performance optimization
* About hiding async complexity

It is about:

* Predictable lifetime
* Ownership of async work
* Safety and correctness

---

## Breaking the Structure

```swift
Task.detached {
    // Unstructured concurrency
}
```

Detached tasks:

* Have no parent
* Are not automatically cancelled
* Must be managed manually

They should be used sparingly and with care.

---

## Why Swift Chose This Model

* Humans reason well about scope
* Humans struggle with unbounded async work
* The compiler can enforce scope rules
* The runtime can enforce cancellation

Structured concurrency aligns async code with how developers already reason about synchronous code.

---

## Real-World Impact (SwiftUI)

```swift
.task {
    await loadData()
}
```

* Task is tied to the view lifecycle
* View disappears → task is cancelled
* Child tasks are cancelled automatically
* No leaked or stale async work

---

## Mastery-Level Insight

> **Structured concurrency transforms async work from “fire-and-forget” into “start-and-finish-within-scope.”**

This eliminates an entire class of lifecycle and cancellation bugs.

---

## Key Takeaways

* Tasks form a hierarchy (task tree)
* Child tasks belong to parent scopes
* Lifetime is enforced by scope
* Cancellation propagates downward
* Async work cannot escape its scope

