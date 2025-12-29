### Background Work Off MainActor

Think like a staff engineer first:

> **The MainActor is a scarce resource.
> Every millisecond you block it steals time from rendering, animations, and user input.**

Your job as an iOS engineer is to **protect the MainActor**.

---

### What the MainActor Is (Reality Check)

* It is a **serial executor**
* It is **tied to the main thread**
* It runs:

  * UI updates
  * SwiftUI rendering
  * Event handling

Rule:

> **Never do expensive work on the MainActor**

---

### What Counts as “Background Work”

Anything that:

* Takes time
* Blocks execution
* Does not immediately update UI

Examples:

* Network calls
* JSON decoding
* Database access
* Image processing
* Crypto / hashing
* Loops and heavy computation

---

### Wrong Pattern (Very Common Junior Mistake)

```swift
@MainActor
func loadProfile() async {
    let data = try await URLSession.shared.data(from: url)
    let user = try JSONDecoder().decode(User.self, from: data.0)
    profile = user
}
```

Why this is bad:

* Entire function runs on MainActor
* JSON decoding blocks UI
* Causes frame drops and jank

---

### Correct Pattern: Split Responsibilities

```swift
func fetchProfile() async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

@MainActor
func updateUI(with user: User) {
    profile = user
}
```

```swift
.task {
    do {
        let user = try await fetchProfile()
        await updateUI(with: user)
    } catch {
        // handle error
    }
}
```

Only UI work touches MainActor.

---

### MainActor Hops Are Explicit and Cheap (But Not Free)

```swift
await MainActor.run {
    title = "Hello"
}
```

Cost:

* Context switch
* Queue scheduling

Rule:

> **Batch UI updates into a single MainActor hop**

Bad:

```swift
await MainActor.run { title = "A" }
await MainActor.run { subtitle = "B" }
```

Good:

```swift
await MainActor.run {
    title = "A"
    subtitle = "B"
}
```

---

### Background Work with Actors (Preferred)

```swift
actor ImageProcessor {
    func resize(_ image: Image) -> Image {
        // heavy CPU work
    }
}
```

```swift
@MainActor
func loadImage() async {
    let resized = await processor.resize(image)
    self.image = resized
}
```

Why this is excellent:

* CPU work off main
* Actor serialization for safety
* Clear ownership

---

### Detached Tasks (Use Sparingly)

```swift
Task.detached(priority: .background) {
    await analytics.track()
}
```

Use only when:

* Work is truly independent
* No UI interaction
* No shared mutable state

Avoid in ViewModels.

---

### Task Priority Matters

```swift
Task(priority: .userInitiated) { }
Task(priority: .background) { }
```

Guidelines:

* UI-triggered work → `.userInitiated`
* Prefetching → `.utility`
* Logging/analytics → `.background`

Priority misuse leads to:

* Starved UI
* Battery drain

---

### Performance & Cancellation (Coupled Concepts)

Bad:

```swift
for _ in 0..<1_000_000 {
    heavyWork()
}
```

Good:

```swift
for _ in 0..<1_000_000 {
    if Task.isCancelled { return }
    heavyWork()
}
```

Why:

* View disappears
* Work should stop
* Save CPU & battery

---

### Async Does NOT Mean Parallel

```swift
await taskA()
await taskB()
```

This is **serial**.

Parallelism:

```swift
async let a = taskA()
async let b = taskB()
await (a, b)
```

Use parallelism carefully:

* CPU contention
* Memory pressure
* Thermal throttling

---

### Performance Pitfall: Accidental MainActor Capture

```swift
@MainActor
class VM {
    func load() async {
        let data = await heavyWork() // ❌ runs on MainActor
    }
}
```

Fix:

```swift
func load() async {
    let data = await heavyWork()
    await MainActor.run {
        updateUI(data)
    }
}
```

---

### Measuring Performance (Senior Practice)

Use:

* Instruments → Time Profiler
* Main Thread Checker
* Swift Concurrency Debug tools

Red flags:

* Long MainActor tasks
* Frequent actor hops
* Excessive task creation

---

### Common Junior Mistakes

❌ Doing JSON decoding on MainActor
❌ Using `DispatchQueue.global()` with async/await
❌ Ignoring cancellation
❌ Excessive `MainActor.run` calls
❌ Assuming async = fast

---

### Mental Model (Memorize This)

* MainActor = UI lifeline
* Heavy work = background
* Actors = safe concurrency
* Fewer hops = better performance
* Cancellation saves battery

---

### Interview-Ready Answer

> “I keep the MainActor minimal by moving all expensive work off it, batching UI updates, and using actors or background tasks. Async doesn’t guarantee performance — ownership and execution context do.”

---

### Junior → Senior Transition Insight

Junior thinks:

> “It works”

Senior thinks:

> “Does it block the MainActor under load?”

---


