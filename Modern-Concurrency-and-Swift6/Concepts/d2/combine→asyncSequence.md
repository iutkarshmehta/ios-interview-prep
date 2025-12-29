### Combine → AsyncSequence (Swift Concurrency)

### Why Combine Needed a Replacement (First Principles)

Combine was designed to:

* Model streams of values over time
* Handle async events
* Chain transformations

But it comes with heavy costs:

* Steep learning curve
* Hard-to-debug pipelines
* Runtime errors instead of compiler errors
* Overkill for many simple async flows

Core issue:

> **Combine models time declaratively but hides execution and ownership.**

AsyncSequence fixes this.

---

### Mental Model Shift (Critical)

| Combine        | AsyncSequence       |
| -------------- | ------------------- |
| Push-based     | Pull-based          |
| Publishers     | AsyncSequence       |
| Subscribers    | for-await loops     |
| Operators      | Control flow        |
| Runtime errors | Compile-time safety |

Key insight:

> **AsyncSequence is just a sequence — values arrive over time.**

---

### When NOT to Migrate from Combine

Do **not** migrate if you need:

* Multiple subscribers
* Advanced operators (debounce, throttle)
* Backpressure control across many consumers

AsyncSequence shines when:

* Single consumer
* Sequential processing
* Clear lifecycle (e.g., ViewModel → View)

---

### Migration Strategy (Always Follow)

1. Identify Combine pipelines
2. Decide if stream is finite or infinite
3. Replace Publisher with AsyncSequence
4. Replace operators with control flow
5. Move consumption into `for await`
6. Handle cancellation explicitly
7. Bind lifecycle (SwiftUI/ViewModel)

---

### Step 1: Publisher → AsyncSequence

### Combine Version

```swift
let publisher = Just(5)
    .map { $0 * 2 }
```

---

### AsyncSequence Version

```swift
let sequence = [5].async.map { $0 * 2 }
```

But real power comes from streams.

---

### Step 2: NotificationCenter Publisher → AsyncSequence

### Combine

```swift
NotificationCenter.default.publisher(for: .didLogin)
    .sink { notification in
        print("Logged in")
    }
    .store(in: &cancellables)
```

Problems:

* Manual cancellation
* Lifetime management

---

### AsyncSequence

```swift
Task {
    for await _ in NotificationCenter.default.notifications(named: .didLogin) {
        print("Logged in")
    }
}
```

Benefits:

* Cancels automatically
* Linear logic
* No cancellables

---

### Step 3: URLSession Publisher → AsyncSequence

### Combine

```swift
URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .sink(receiveValue: { data in
        print(data)
    })
    .store(in: &cancellables)
```

---

### async/await (No Stream Needed)

```swift
let (data, _) = try await URLSession.shared.data(from: url)
print(data)
```

Rule:

> **If it’s a single value, AsyncSequence is unnecessary.**

---

### Step 4: Timer Publisher → AsyncSequence

### Combine

```swift
Timer.publish(every: 1, on: .main, in: .common)
    .autoconnect()
    .sink { _ in
        print("Tick")
    }
```

---

### AsyncSequence

```swift
for await _ in Timer.publish(every: 1, tolerance: 0.1).values {
    print("Tick")
}
```

Cleaner, cancellable, readable.

---

### Step 5: Custom Publisher → AsyncStream

### Combine Custom Publisher

Complex, error-prone, verbose.

---

### AsyncStream Replacement

```swift
let stream = AsyncStream<Int> { continuation in
    for i in 1...5 {
        continuation.yield(i)
    }
    continuation.finish()
}
```

Consumption:

```swift
for await value in stream {
    print(value)
}
```

---

### Step 6: Error Handling (Big Improvement)

### Combine

```swift
.publisher
    .sink(receiveCompletion: { completion in
        if case .failure(let error) = completion {
            print(error)
        }
    }, receiveValue: { value in
        print(value)
    })
```

---

### AsyncThrowingStream

```swift
let stream = AsyncThrowingStream<Int, Error> { continuation in
    if Bool.random() {
        continuation.yield(1)
    } else {
        continuation.finish(throwing: MyError.failed)
    }
}
```

Consumption:

```swift
do {
    for try await value in stream {
        print(value)
    }
} catch {
    print(error)
}
```

---

### Step 7: SwiftUI ViewModel Migration

### Combine ViewModel

```swift
class VM: ObservableObject {
    @Published var value: Int = 0
    var cancellables = Set<AnyCancellable>()

    init() {
        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { _ in
                self.value += 1
            }
            .store(in: &cancellables)
    }
}
```

Problems:

* Memory management
* Retain cycles risk

---

### AsyncSequence ViewModel

```swift
@MainActor
class VM: ObservableObject {
    @Published var value = 0

    func start() {
        Task {
            for await _ in Timer.publish(every: 1).values {
                value += 1
            }
        }
    }
}
```

Cleaner and safer.

---

### Step 8: Cancellation Behavior (Huge Win)

Combine:

* Must store & cancel manually

AsyncSequence:

* Cancels when task is cancelled
* SwiftUI `.task` cancels automatically

Rule:

> **Cancellation propagates naturally.**

---

### Step 9: Performance Considerations

* AsyncSequence is lightweight
* Avoid unbounded streams
* Avoid heavy work inside for-await
* Prefer `await` suspension over blocking
* Measure long-running streams

---

### Step 10: Common Migration Mistakes

* Recreating Combine operators manually
* Forgetting to finish streams
* Multiple consumers on AsyncSequence
* Blocking MainActor in loops
* Overusing AsyncStream

---

### Debugging AsyncSequence

Tools:

* Xcode Thread Sanitizer
* Instruments → Time Profiler
* Logging suspension points

Watch for:

* Never-ending loops
* Forgotten cancellation
* MainActor blocking

---

### Interview Cross-Questions

**Q: Push vs Pull model difference?**
A: Combine pushes values; AsyncSequence pulls values when ready.

**Q: Can AsyncSequence replace Combine fully?**
A: No — Combine is better for multi-subscriber reactive systems.

**Q: How does backpressure work in AsyncSequence?**
A: Consumer controls demand naturally by awaiting.

---

### One-Line Senior Summary

> “Combine models streams declaratively; AsyncSequence models them sequentially with ownership and safety.”

---

### Junior → Senior Mental Shift

Junior thinks:

> “Replace publisher with AsyncSequence.”

Senior thinks:

> “Who consumes this stream, how long does it live, and who cancels it?”

---

