### AsyncSequence

As a staff engineer, think of **AsyncSequence as the async counterpart of Sequence**.
It models **a stream of values that arrive over time**, instead of all at once.

Core idea:

* `Sequence` → pull-based, synchronous
* `AsyncSequence` → pull-based, **asynchronous**
* Each element may require **suspension** before it’s available

Conceptual difference:

```text
Sequence:     value → value → value → end
AsyncSequence: await → value → await → value → await → end
```

Swift definition (simplified):

```swift
protocol AsyncSequence {
    associatedtype Element
    associatedtype AsyncIterator: AsyncIteratorProtocol
    func makeAsyncIterator() -> AsyncIterator
}
```

You rarely implement this yourself — Swift gives you **powerful built-ins**.

---

### Built-in AsyncSequence Examples

#### AsyncSequence from Swift Concurrency

```swift
let numbers = [1, 2, 3]

Task {
    for await number in numbers.async {
        print(number)
    }
}
```

#### URLSession Bytes (Real iOS Example)

```swift
let (bytes, _) = try await URLSession.shared.bytes(from: url)

for await byte in bytes {
    print(byte)
}
```

This is true streaming — data arrives incrementally.

---

### for await Loops

The **only correct way** to consume an AsyncSequence.

```swift
for await element in asyncSequence {
    // process element
}
```

Key rules:

* Must be inside an `async` context
* Suspends between elements
* Automatically stops when the sequence finishes
* Cancellation-aware

Equivalent mental model:

```swift
while let element = await iterator.next() {
    process(element)
}
```

---

### AsyncStream

`AsyncStream` is how you **create your own AsyncSequence**.

Use case:

* Bridging callback-based APIs
* Emitting values over time
* Modeling events (timers, notifications, sensors)

Basic example:

```swift
let stream = AsyncStream<Int> { continuation in
    continuation.yield(1)
    continuation.yield(2)
    continuation.yield(3)
    continuation.finish()
}
```

Consumption:

```swift
Task {
    for await value in stream {
        print(value)
    }
}
```

---

### AsyncStream with Time-Based Emission

```swift
let timerStream = AsyncStream<Date> { continuation in
    let timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
        continuation.yield(Date())
    }

    continuation.onTermination = { _ in
        timer.invalidate()
    }
}
```

This models a **push-based source** using a pull-based consumer.

---

### Continuation Lifecycle

Important methods:

```swift
continuation.yield(value)
continuation.finish()
continuation.onTermination = { reason in }
```

Termination reasons:

* Consumer finished iteration
* Task cancelled
* Stream finished manually

This makes AsyncStream **resource-safe**.

---

### AsyncThrowingStream

Same as AsyncStream, but supports **errors**.

Use when:

* The stream can fail
* Errors are part of the contract

Definition:

```swift
AsyncThrowingStream<Element, Error>
```

---

### AsyncThrowingStream Example

```swift
enum NetworkError: Error {
    case disconnected
}

let stream = AsyncThrowingStream<Int, Error> { continuation in
    continuation.yield(1)
    continuation.yield(2)
    continuation.finish(throwing: NetworkError.disconnected)
}
```

Consumption:

```swift
Task {
    do {
        for try await value in stream {
            print(value)
        }
    } catch {
        print("Stream failed:", error)
    }
}
```

Key difference:

* `for await` → AsyncSequence
* `for try await` → AsyncThrowingSequence

---

### Bridging Callback APIs (Real-World Pattern)

Legacy API:

```swift
func fetchUpdates(handler: @escaping (Int?, Error?) -> Void)
```

Bridge to AsyncThrowingStream:

```swift
func updatesStream() -> AsyncThrowingStream<Int, Error> {
    AsyncThrowingStream { continuation in
        fetchUpdates { value, error in
            if let value {
                continuation.yield(value)
            } else if let error {
                continuation.finish(throwing: error)
            }
        }
    }
}
```

Now consumed safely:

```swift
for try await update in updatesStream() {
    print(update)
}
```

---

### Cancellation Behavior (Critical Topic)

```swift
Task {
    for await value in stream {
        print(value)
    }
}
```

If the task is cancelled:

* Iteration stops
* `onTermination` is called
* Resources must be cleaned up

This is **structured concurrency** in action.

---

### Backpressure and Buffering

AsyncStream supports buffering policies:

```swift
AsyncStream(
    bufferingPolicy: .bufferingNewest(5)
) { continuation in
    // yields here
}
```

Options:

* `.unbounded`
* `.bufferingOldest(n)`
* `.bufferingNewest(n)`

This prevents memory leaks under high-frequency emissions.

---

### AsyncSequence vs Combine

| AsyncSequence          | Combine        |
| ---------------------- | -------------- |
| Language-level         | Framework      |
| Structured concurrency | Reactive       |
| Cancellation built-in  | Manual         |
| Backpressure aware     | Complex        |
| Easier reasoning       | More operators |

AsyncSequence is **simpler and safer** for most async flows.

---

### Senior Engineer Mental Model

* **AsyncSequence** → abstraction
* **AsyncStream** → value stream
* **AsyncThrowingStream** → fallible stream
* **for await** → consumption contract
* **Cancellation** → automatic and safe

---

### Common Mistakes (Interview Gold)

* Forgetting to call `finish()`
* Ignoring `onTermination`
* Using AsyncStream for one-shot values (overkill)
* Not handling cancellation
* Emitting values after termination

---

### When to Use What

* One async value → `async/await`
* Multiple async values → `AsyncSequence`
* Manual emission → `AsyncStream`
* Failure possible → `AsyncThrowingStream`

---
