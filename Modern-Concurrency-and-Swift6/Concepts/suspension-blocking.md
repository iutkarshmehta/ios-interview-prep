
### Suspension vs Blocking

### What Is Blocking?
Blocking means a **thread waits** and cannot do any other work.
- The thread stops executing
- The CPU sits idle
- No other tasks can use that thread

Example:
```swift
let data = try Data(contentsOf: url) // blocks the thread
````

### Why Blocking Is Bad

* Threads are heavy and limited
* Blocking wastes system resources
* Leads to UI freezes and poor performance
* Blocking the main thread makes apps unresponsive

### What Is Suspension?

Suspension means a **function pauses**, not the thread.

* The function stops executing
* The thread is released back to the system
* The thread runs other tasks
* The function resumes later

Example:

```swift
let data = await fetchData()
```

### Core Difference

**Blocking pauses a thread**
**Suspension pauses a function**

This is the most important distinction in Swift Concurrency.

### Visual Intuition — Blocking

* Thread is stuck waiting
* No other work can run on that thread
* System must create more threads

### Visual Intuition — Suspension

* Function suspends at `await`
* Thread executes other tasks
* Suspended function resumes later

### Why `await` Exists

`await` marks a suspension point.

* Execution may pause
* Time may pass
* Thread may change
* State may change

Swift forces this to be explicit.

### Suspension on the Main Thread

```swift
@MainActor
func loadData() async {
    let data = await fetchData()
    updateUI(data)
}
```

* Main thread stays responsive
* UI does not freeze
* Safe and efficient

### What Happens Under the Hood

* Function state is saved
* Continuation is stored
* Thread is reused
* Execution resumes when data is ready

### Why GCD Still Blocked Threads

* GCD executes closures on threads
* No concept of function suspension
* Blocking is common and easy to misuse
* Deadlocks are possible

### Suspension Enables Structured Concurrency

* Parent tasks don’t block
* Child tasks run concurrently
* Cancellation propagates correctly
* Lifetime is enforced by scope

### Common Misconceptions

* `await` blocks the thread ❌
* async means background execution ❌
* suspension is slower ❌
* main thread cannot suspend ❌

### Staff-Level Rule

**Blocking wastes threads. Suspension scales work.**

### Final Mental Model

* Threads are system resources
* Functions are logical units of work
* Swift suspends functions to protect threads
* This enables safe, scalable concurrency


