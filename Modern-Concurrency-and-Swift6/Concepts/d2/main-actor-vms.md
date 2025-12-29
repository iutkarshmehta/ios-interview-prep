### `@MainActor` ViewModels

### Why `@MainActor` ViewModels Exist (First Principles)

Two hard rules of iOS:

1. **UI must be updated on the main thread**
2. **Async work must not block the UI**

Before Swift Concurrency:

* Developers manually used `DispatchQueue.main.async`
* Easy to forget
* Easy to introduce race conditions

Swift Concurrency solution:

> **Make UI state live on a single actor: the MainActor**

This is exactly what `@MainActor` ViewModels do.

---

### What `@MainActor` Actually Means

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var name: String = ""
}
```

Meaning:

* All properties are **MainActor-isolated**
* All methods run on the **main actor**
* Any access from background tasks must `await`

Important:

> `@MainActor` is **not a thread**
> It is a **serial executor tied to the main thread**

---

### Why ViewModels Are the Right Place

SwiftUI views:

* Are recreated frequently
* Should stay lightweight
* Should not own business logic

ViewModels:

* Are long-lived
* Own UI state
* Coordinate async work

So:

> **UI state + concurrency logic belongs in a `@MainActor` ViewModel**

---

### Basic Example (Correct Pattern)

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var username: String = ""

    func loadProfile() async {
        let name = await fetchNameFromServer()
        username = name
    }
}
```

Usage in view:

```swift
struct ProfileView: View {
    @StateObject private var vm = ProfileViewModel()

    var body: some View {
        Text(vm.username)
            .task {
                await vm.loadProfile()
            }
    }
}
```

Why this is perfect:

* `.task` may run off-main
* `vm.loadProfile()` hops to MainActor safely
* No thread bugs
* No manual dispatching

---

### What Happens Under the Hood (Execution Flow)

1. View appears
2. `.task` starts on a background executor
3. `await vm.loadProfile()` called
4. Swift **hops to MainActor**
5. UI state updates safely
6. View re-renders

All enforced by the compiler.

---

### What Happens If You Forget `@MainActor`

```swift
final class BadViewModel: ObservableObject {
    @Published var title = ""
}
```

```swift
.task {
    vm.title = "Hello"
}
```

❌ Compile-time error (strict concurrency):

> Publishing changes from background threads is not allowed

Without `@MainActor`:

* UI updates are unsafe
* Races are possible
* Bugs appear intermittently

---

### Alternative: Annotating Methods Instead of Class

```swift
final class PartialViewModel: ObservableObject {
    @Published var title = ""

    @MainActor
    func updateTitle(_ text: String) {
        title = text
    }
}
```

When to use:

* Mixed UI + non-UI logic
* Advanced cases only

For juniors:

> Prefer annotating the **entire ViewModel**

---

### Async Work Inside `@MainActor` ViewModels

Important rule:

> **Never do heavy work on MainActor**

Bad:

```swift
@MainActor
func load() async {
    let data = try await heavyNetworkCall() // ❌ blocks UI
    title = data
}
```

Correct:

```swift
func load() async {
    let data = try await heavyNetworkCall()
    await MainActor.run {
        title = data
    }
}
```

Even better:

```swift
func load() async {
    let data = try await repository.fetch()
    await updateUI(data)
}

@MainActor
func updateUI(_ data: String) {
    title = data
}
```

---

### ViewModel + Actor Collaboration (Senior Pattern)

```swift
actor UserRepository {
    func fetchName() async -> String {
        "Utkarsh"
    }
}
```

```swift
@MainActor
final class VM: ObservableObject {
    let repo = UserRepository()
    @Published var name = ""

    func load() async {
        let result = await repo.fetchName()
        name = result
    }
}
```

Actors:

* Own data
* Do background-safe work

ViewModel:

* Orchestrates
* Updates UI

---

### Cancellation Behavior

```swift
.task {
    await vm.load()
}
```

If view disappears:

* Task is cancelled
* Cancellation propagates into `load()`
* Any `await` inside responds to cancellation

Best practice:

```swift
func load() async {
    guard !Task.isCancelled else { return }
    let data = await fetch()
    name = data
}
```

---

### Common Junior Mistakes (Very Important)

❌ Making ViewModels value types
❌ Using `DispatchQueue.main.async` inside `@MainActor`
❌ Using `Task.detached`
❌ Performing networking on MainActor
❌ Updating UI from repositories

---

### Interview Cross-Questions (Be Ready)

**Q: Why annotate the whole ViewModel with `@MainActor`?**
A: Because it owns UI state, and Swift enforces thread safety at compile time.

**Q: Does `@MainActor` guarantee main thread execution?**
A: Yes, via a serial executor tied to the main thread.

**Q: Can async methods exist in `@MainActor` classes?**
A: Yes, but heavy work must not block the actor.

---

### Mental Model (Memorize This)

* ViewModels own UI state
* UI state lives on MainActor
* Views trigger async work
* Actors do background work
* Compiler enforces correctness

---

### One-Line Senior Summary

> “`@MainActor` ViewModels centralize UI state and concurrency, giving compile-time thread safety and predictable SwiftUI behavior.”

---
