
### 1. `nonisolated`

### Why `nonisolated` exists

Actors protect **mutable state**.
But not everything inside an actor actually needs protection.

Sometimes you want:

* Utility logic
* Pure functions
* Constants
* Protocol conformance without `async`

That’s where `nonisolated` comes in.

---

### What `nonisolated` means

When you mark a member `nonisolated`, you are saying:

> “This code does **not** depend on actor state and is safe to call from anywhere.”

```swift
actor IDGenerator {
    let prefix = "ID"

    nonisolated func format(_ number: Int) -> String {
        "\(prefix)-\(number)"
    }
}
```

Key rules:

* Cannot access actor-isolated state
* Can be called **synchronously**
* Must be thread-safe by design

The compiler enforces this strictly.

---

### Why this matters in real code

Without `nonisolated`:

* Everything on an actor is `async`
* Protocol conformance becomes painful
* Performance suffers from unnecessary awaits

With `nonisolated`:

* Clean APIs
* Clear intent
* Better performance

---

### Common mistake

```swift
nonisolated func log() {
    print(self.prefix) // ❌ error
}
```

The compiler blocks this because:

* `prefix` belongs to actor isolation
* `nonisolated` code cannot touch it

---

### 2. Global Actors

### The problem global actors solve

Sometimes you want:

* Many types and functions
* All protected by the **same execution context**
* Without passing actor instances everywhere

Classic example:

* UI code must run on main thread

This is where **global actors** come in.

---

### What is a global actor?

A **global actor** is:

* A globally shared actor instance
* Identified by an attribute
* Used to isolate functions, types, or properties

```swift
@globalActor
actor AnalyticsActor {
    static let shared = AnalyticsActor()
}
```

Now you can do:

```swift
@AnalyticsActor
func trackEvent() {
    // isolated to AnalyticsActor
}
```

Everything annotated:

* Runs on the same actor
* Has serialized access
* Gets compiler enforcement

---

### Why global actors are powerful

They:

* Replace global serial queues
* Prevent accidental thread misuse
* Make architectural boundaries explicit

You move from:

> “Hope this runs on the right queue”

to:

> “The compiler guarantees it”

---

### 3. `@MainActor`

### What `@MainActor` really is

`@MainActor` is just a **predefined global actor**.

```swift
@MainActor
final class ViewModel {
    var title: String = ""
}
```

Guarantee:

* All access happens on the main thread
* UI updates are safe by default

No more:

* `DispatchQueue.main.async`
* Guesswork
* UI race conditions

---

### How `@MainActor` works

When you call a `@MainActor` method:

* Swift automatically hops to the main thread
* Suspends current task if needed

```swift
await viewModel.updateUI()
```

This hop is:

* Safe
* Automatic
* Compiler-verified

---

### Applying `@MainActor`

### On functions

```swift
@MainActor
func updateUI() { }
```

### On types

```swift
@MainActor
class ViewModel { }
```

### On properties

```swift
@MainActor
var currentUser: User
```

Scope matters:

* Type-level applies to all members
* Member-level overrides

---

### Interaction with background work

```swift
func loadData() async {
    let data = await fetchData()

    await MainActor.run {
        updateUI(data)
    }
}
```

`MainActor.run`:

* Explicit hop to main thread
* Scoped and clear
* Preferable to GCD

---

### Staff-Level Design Guidance

### Use `nonisolated` when:

* Code is pure
* No shared state
* You want sync access

### Use global actors when:

* You have a shared subsystem
* Serialization matters
* You want architectural clarity

### Use `@MainActor` when:

* UI is involved
* State must be main-thread-bound
* You want compiler-enforced safety

---

### Final Mental Model

* Actors protect state
* `nonisolated` opts out safely
* Global actors define shared execution contexts
* `@MainActor` protects the UI
* Compiler replaces discipline

---

