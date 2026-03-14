

---

# Swift Concurrency Interview Notes

# Question 5: What is Cooperative Cancellation?

## Definition

In Swift concurrency, **task cancellation is cooperative**.

> Cancelling a task **does not forcefully stop it**.
> Instead, the task is **marked as cancelled**, and the task itself must check this state and exit.

---

# How Cancellation Works

Example:

```swift
let task = Task {
    await work()
}

task.cancel()
```

When `cancel()` is called:

```text
Task state → cancelled
```

But the task **keeps running** unless it checks for cancellation.

---

# How Tasks Check for Cancellation

## 1. `Task.isCancelled`

Manually check cancellation state.

```swift
func process() async {

    for i in 0...100 {

        if Task.isCancelled {
            print("Task cancelled")
            return
        }

        await doWork()
    }
}
```

Here the task **cooperatively exits**.

---

## 2. `Task.checkCancellation()`

Throws an error if the task is cancelled.

```swift
func process() async throws {

    try Task.checkCancellation()

    await doWork()
}
```

If cancelled:

```text
CancellationError is thrown
```

---

# Cancellation-Aware Suspension Points

Some Swift APIs automatically check for cancellation.

Example:

```swift
try await Task.sleep(nanoseconds: 5_000_000_000)
```

If the task is cancelled during sleep:

```text
CancellationError thrown
```

Example:

```swift
let task = Task {
    try await Task.sleep(nanoseconds: 5_000_000_000)
    print("Finished")
}

task.cancel()
```

Output:

```text
"Finished" will NOT print
```

because `Task.sleep` throws `CancellationError`.

---

# Infinite Loop Example

This task **will not stop** even if cancelled:

```swift
Task {
    while true {
        print("Working")
    }
}
```

Why?

* No suspension point
* No cancellation check

---

# Correct Way

```swift
Task {
    while !Task.isCancelled {
        print("Working")
    }
}
```

or

```swift
Task {
    while true {
        try Task.checkCancellation()
        await doWork()
    }
}
```

---

# Why Swift Uses Cooperative Cancellation

Forcefully killing tasks can cause:

* inconsistent state
* resource leaks
* incomplete operations

So Swift allows the task to **exit safely and clean up resources**.

---

# Interview One-Line Answer

> Swift uses cooperative cancellation, meaning cancelling a task only marks it as cancelled, and the task must check for cancellation using `Task.isCancelled` or `Task.checkCancellation()` and terminate itself.

---

