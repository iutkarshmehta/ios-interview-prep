### The Real Problem GCD and async/await Try to Solve

Programs need to do **multiple things at the same time** without:

* blocking the UI
* freezing the app
* corrupting shared state

This problem is called **concurrency**.

Concurrency is about **structuring work**, not about threads.

---

### What GCD Really Is (Conceptually)

GCD is a system where you:

* submit **units of work**
* to **queues**
* and the system decides **when and on which thread** they run

You do **not** manage threads.
You describe *what* should run and *where*.

---

### Dispatch Queues: The Core Abstraction

A dispatch queue is:

> A line of tasks waiting to be executed.

Two important properties:

* **Serial**: one task at a time
* **Concurrent**: multiple tasks can run simultaneously

---

### The Main Queue (Special Case)

The main queue:

* is serial
* runs on the main thread
* handles UI work

Rule:

> UI work must happen on the main queue.

Breaking this rule leads to:

* crashes
* undefined behavior
* visual bugs

---

### Background Queues

Background queues:

* run tasks off the main thread
* are used for:

  * networking
  * parsing
  * heavy computation

You don’t choose threads — you choose **priority**.

---

### The Classic GCD Pattern (Mental Model)

```swift
DispatchQueue.global().async {
    let data = fetchData()

    DispatchQueue.main.async {
        updateUI(data)
    }
}
```

Conceptually:

* “Do heavy work somewhere else”
* “When done, come back to UI”

This pattern works, but has **hidden problems**.

---

### Problems With GCD (Why It Doesn’t Scale Well)

GCD has no built-in concept of:

* task ownership
* cancellation
* parent/child relationships
* structured lifetimes

As code grows:

* callbacks nest
* error handling fragments
* memory ownership becomes unclear

This leads to:

* callback hell
* leaked objects
* un-cancellable work

---

### Closures and GCD (Critical Connection)

Every `async {}` block is a **closure**.

That closure:

* may run later
* may run long after the surrounding object should disappear

So GCD naturally introduces **lifetime extension** bugs.

---

### The Core Limitation of GCD

GCD answers:

> “Where should this work run?”

It does **not** answer:

> “Who owns this work?”
> “When should this work stop?”

That gap is why modern concurrency exists.

---

### What async/await Is (Conceptually)

async/await is **not just syntax sugar**.

It introduces:

* structured concurrency
* clear task lifetimes
* cancellation propagation

The key idea:

> Work should have a clear beginning, scope, and end.

---

### Tasks: The Fundamental Unit in async/await

A `Task` represents:

* a unit of asynchronous work
* with a defined lifetime
* that can be cancelled

Unlike GCD blocks:

* tasks know when they should stop
* tasks know their parent

---

### Structured Concurrency (The Big Shift)

Structured concurrency means:

* child tasks cannot outlive their parent
* work is scoped
* lifetimes are predictable

This prevents “runaway background work”.

---

### async/await Example (Conceptual View)

```swift
Task {
    let data = try await fetchData()
    updateUI(data)
}
```

Think of this as:

* “Start a piece of work”
* “Pause until data arrives”
* “Continue in order”

No nesting.
No callbacks.
Clear flow.

---

### Suspension, Not Blocking (Important Concept)

When you `await`:

* the thread is **not blocked**
* execution is suspended
* the thread is reused for other work

This is why async/await scales efficiently.

---

### MainActor: The UI Safety Boundary

`@MainActor` is a **contract**.

It means:

> “This code must run on the main thread.”

The compiler enforces this.

This replaces:

* manual `DispatchQueue.main.async`
* guesswork

---

### async/await Does NOT Remove Memory Issues

This is critical.

```swift
Task {
    await self.loadData()
}
```

Conceptually:

* Task owns the closure
* closure owns `self`
* task may outlive `self`

Memory issues still exist.

---

### Why `[weak self]` Is Still Needed

Because async/await changes **syntax**, not **ownership rules**.

Correct pattern:

```swift
Task { [weak self] in
    await self?.loadData()
}
```

The same lifetime reasoning applies.

---

### Cancellation: The Missing Piece in GCD

With async/await:

* tasks can be cancelled
* cancellation propagates
* work can stop cooperatively

GCD has no native cancellation model.

---

### Concurrency Is About Ownership, Not Threads

Threads are an implementation detail.

The real questions are:

* Who started this work?
* Who owns it?
* When should it stop?
* What happens if the UI disappears?

async/await makes these questions explicit.

---

### The Unifying Mental Model

GCD:

* fire-and-forget work submission

async/await:

* scoped, owned, cancellable work

Same goal.
Different discipline.

---

### One-Line Deep Understanding

> GCD schedules work; async/await structures work.

If this sentence makes sense, you understand the shift.

---

### Pause Here

Before we move on, try to explain:

* why async/await still needs weak references
* why cancellation matters
* why structure matters more than threads

---

### What Cancellation Actually Means (Real World Meaning)

Cancellation means:

> “This work is no longer needed. Stop as soon as it is safe.”

Important:

* Cancellation is **cooperative**
* The system does not kill work forcefully
* Your code must **check and respect cancellation**

---

### Real World Example 1: User Leaves a Screen During Network Call

Scenario:

* User opens Profile screen
* Network request starts
* User presses Back
* The result is no longer needed

---

### GCD Version (No Real Cancellation)

```swift
DispatchQueue.global().async {
    let data = fetchProfile()

    DispatchQueue.main.async {
        self.updateUI(data)
    }
}
```

Problems:

* Work continues even after screen is gone
* Memory and battery wasted
* Potential crash if `self` is gone
* No way to stop the work

You can only **ignore** the result — not stop the work.

---

### async/await Version With Cancellation

```swift
class ProfileViewModel {
    var task: Task<Void, Never>?

    func loadProfile() {
        task = Task {
            let data = try await fetchProfile()
            await MainActor.run {
                updateUI(data)
            }
        }
    }

    func cancel() {
        task?.cancel()
    }
}
```

Now:

* Work has an owner
* Work can be cancelled
* Lifecycle is controlled

---

### What Happens Internally on Cancellation

When `cancel()` is called:

* Task is marked as cancelled
* `Task.isCancelled` becomes true
* `await` points can throw `CancellationError`

Nothing is force-killed.
Your code decides how to respond.

---

### Respecting Cancellation Properly

```swift
func fetchProfile() async throws -> Profile {
    try Task.checkCancellation()
    let data = try await apiCall()
    try Task.checkCancellation()
    return data
}
```

This ensures:

* Work stops early
* Resources are freed
* No useless processing

---

### Real World Example 2: Search-as-You-Type

Scenario:

* User types fast
* Every keystroke triggers a search
* Old searches must stop

---

### Wrong Approach (GCD)

```swift
DispatchQueue.global().async {
    let result = search(query)
    DispatchQueue.main.async {
        show(result)
    }
}
```

Problems:

* Old queries still run
* Results may arrive out of order
* UI shows stale data

---

### Correct async/await Approach

```swift
var searchTask: Task<Void, Never>?

func performSearch(_ query: String) {
    searchTask?.cancel()

    searchTask = Task {
        try await Task.sleep(nanoseconds: 300_000_000)
        let result = try await search(query)
        await MainActor.run {
            show(result)
        }
    }
}
```

Now:

* Only latest task runs
* Older work stops
* UI stays consistent

---

### Why Cancellation Is Impossible to Model Cleanly With GCD

GCD:

* has no task identity
* has no ownership
* has no parent-child relationship

Once submitted:

> the work will run — no questions asked

Cancellation becomes:

* ad-hoc flags
* manual checks
* fragile logic

---

### Why Structure Matters More Than Threads

Threads answer:

> “Where does code run?”

Structure answers:

> “Why does this code exist, and when should it stop?”

Correct programs need **both**, but structure is more important.

---

### Example: Without Structure

```swift
DispatchQueue.global().async {
    doA()
    DispatchQueue.global().async {
        doB()
        DispatchQueue.global().async {
            doC()
        }
    }
}
```

Questions you cannot answer:

* Who owns `doC`?
* When should `doB` stop?
* What if the UI disappears?
* How do you cancel everything?

---

### Same Logic With Structure

```swift
Task {
    async let a = doA()
    async let b = doB()
    let c = await doC()
}
```

Now:

* All work has a common scope
* Cancellation propagates
* Failure is contained
* Lifetime is predictable

---

### Parent–Child Relationship (Critical Insight)

In structured concurrency:

* Parent starts work
* Children cannot outlive parent
* Parent controls cancellation

This prevents:

* runaway background work
* memory leaks
* inconsistent state

---

### Why This Matters for iOS Apps

iOS apps are:

* UI-driven
* lifecycle-heavy
* interruption-prone

Users:

* navigate quickly
* background apps
* kill apps anytime

Unstructured work leads to:

* wasted CPU
* memory leaks
* bugs that are hard to reproduce

---

### The Core Philosophy Shift

Old model:

> “Start work and hope it finishes correctly.”

New model:

> “Start work only while it is needed.”

This is the real power of async/await.

---

### One-Line Deep Truth

> Threads execute code.
> Structure gives that code meaning and limits.

---

### Final Mental Model

Every async operation should answer:

* Who started me?
* Who owns me?
* When should I stop?
* What happens if my owner disappears?

---


