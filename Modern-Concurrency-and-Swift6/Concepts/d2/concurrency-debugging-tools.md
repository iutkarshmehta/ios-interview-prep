### Why Concurrency Debugging Tools Exist

Concurrency bugs are:

* Non-deterministic
* Timing-dependent
* Often invisible in normal debugging

Symptoms you’ll see in real apps:

* Random crashes
* UI freezes
* Stale or corrupted data
* Bugs that disappear when you add logs

Goal:

> **Understand who ran where, when, and what blocked what**

---

### Mental Model for Debugging Concurrency

Concurrency bugs are **time bugs**, not logic bugs.

You debug:

* Execution order
* Task lifetimes
* Actor ownership
* Blocking points
* Cancellation paths

---

### Thread Sanitizer (TSAN)

TSAN detects:

* Data races
* Unsynchronized shared memory access

It is your **first line of defense**.

---

### How to Enable Thread Sanitizer

Xcode → Scheme → Run → Diagnostics → ✅ Thread Sanitizer

Notes:

* Debug builds only
* Slows execution
* Extremely accurate

---

### TSAN Example: Data Race

```swift
var counter = 0

DispatchQueue.global().async {
    counter += 1
}

DispatchQueue.global().async {
    counter += 1
}
```

TSAN reports:

* Race on `counter`
* Stack traces of both writes

---

### TSAN with Swift Concurrency

```swift
var name = "A"

Task {
    name = "B"
}
```

With strict concurrency:

* Compile-time error
  Without it:
* TSAN catches runtime race

TSAN complements the compiler.

---

### Main Thread Checker

Main Thread Checker detects:

* UI updates from background threads
* UIKit/SwiftUI violations

---

### Main Thread Checker Example

```swift
Task {
    label.text = "Hello" // ❌
}
```

Runtime warning:

> UI API called on a background thread

---

### Correct Pattern

```swift
await MainActor.run {
    label.text = "Hello"
}
```

---

### Xcode Debugger: Thread View

Xcode Debug Navigator shows:

* All threads
* Call stacks
* Blocked threads

Use it to identify:

* Deadlocks
* Long-running tasks
* Actor contention

---

### Debugging Actor Contention

Symptoms:

* App “feels async” but is slow
* UI waits for actor responses

In debugger:

* Many tasks waiting on same actor
* Long actor call stacks

Fix:

* Reduce actor scope
* Move heavy work off actor

---

### Instruments: Time Profiler

Time Profiler answers:

> **Where is time actually spent?**

Use it to detect:

* MainActor blocking
* Heavy CPU work
* Excessive task creation

---

### Time Profiler Workflow

1. Run Instruments → Time Profiler
2. Interact with app
3. Look for:

   * Long Main Thread blocks
   * Frequent context switches

---

### Instruments: Thread States

Thread States show:

* Running
* Waiting
* Blocked

Use it to:

* Detect deadlocks
* Identify priority inversion

---

### Detecting Priority Inversion

Symptoms:

* UI stutters
* User-initiated work waits

In Instruments:

* High-priority thread waiting
* Low-priority thread running

Fix:

* Align task priorities
* Avoid blocking UI tasks

---

### Swift Concurrency Debug Logs

Enable concurrency logs:

Environment Variable:

```
SWIFT_CONCURRENCY_LOG_LEVEL=3
```

Shows:

* Task creation
* Task cancellation
* Executor hops

Use when:

* Tasks mysteriously cancel
* Async flows feel unpredictable

---

### Logging Task Identity

```swift
print(Task.currentPriority)
print(Task.isCancelled)
```

Use this to:

* Debug cancellation
* Track task execution paths

---

### Debugging Cancellation Bugs

Common bug:

```swift
.task {
    while true {
        await pollServer()
    }
}
```

Debugging signal:

* Task never ends
* CPU usage high

Fix:

```swift
while !Task.isCancelled {
    await pollServer()
}
```

---

### Debugging AsyncSequence Issues

If UI freezes:

* Check `for await` loops
* Ensure cancellation is handled
* Verify `onTermination` is implemented

---

### Memory Debugger + Concurrency

Use Memory Graph Debugger to find:

* Retained tasks
* Leaked view models
* Tasks capturing `self`

Common leak:

```swift
Task {
    await self.load()
}
```

Fix:

* Ensure task cancellation
* Use `.task` instead of detached tasks

---

### Common Junior Debugging Mistakes

* Ignoring TSAN warnings
* Debugging concurrency with print statements
* Assuming async = parallel
* Forgetting cancellation
* Overusing detached tasks

---

### Senior Engineer Debugging Rules

* Trust tools over intuition
* Debug execution order, not code order
* Measure before optimizing
* Fix ownership, not symptoms
* Assume cancellation always happens

---

### Interview-Ready Summary

> “I debug concurrency using TSAN for data races, Main Thread Checker for UI violations, Instruments for performance and contention, and Swift concurrency logs to understand task lifetimes and cancellation.”

---

### What This Prepares You For

You’re now ready to:

* Debug production race conditions
* Diagnose UI freezes
* Fix actor contention
* Reason about task cancellation
* Answer senior-level interview questions

---
