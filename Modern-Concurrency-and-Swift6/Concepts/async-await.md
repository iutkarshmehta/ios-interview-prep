
# Swift `async / await` — Basics

## Why `async / await` Exists

Before Swift Concurrency, async code relied on callbacks:

```swift
fetchUser { user in
    fetchPosts(user) { posts in
        updateUI(posts)
    }
}
````

Problems:

* Callback pyramids
* Scattered error handling
* Hard-to-follow control flow
* Manual lifetime and cancellation
* Difficult reasoning

**Goal:** Write async code that looks synchronous without blocking threads.

---

## What `async` Means

### Definition

`async` indicates that:

* A function can **suspend and resume**
* It does **not block a thread**
* It does **not imply background execution**

```swift
func fetchUser() async -> User
```

This tells the compiler that calling this function may require suspension.

---

## What `await` Means

### Definition

`await` marks a **suspension point**.

```swift
let user = await fetchUser()
```

At this point:

* The function may pause
* The thread is released
* Execution resumes later with the result

---

## Suspension vs Blocking

### Blocking (Old Model)

* Thread waits
* Wastes system resources
* Limits scalability

### Suspension (Swift Concurrency)

* Function pauses
* Thread is reused
* Highly efficient

> **Async / await suspends functions, not threads.**

---

## Execution Flow Example

```swift
func loadData() async {
    print("Start")
    let user = await fetchUser()
    print("User fetched")
}
```

Execution steps:

1. Prints `Start`
2. Suspends at `await`
3. Thread is reused elsewhere
4. Function resumes when data is ready
5. Prints `User fetched`

---

## Async Code Is Sequential by Default

```swift
let user = await fetchUser()
let posts = await fetchPosts()
```

* Runs sequentially
* No parallelism by default

---

## Running Async Work Concurrently

```swift
async let user = fetchUser()
async let posts = fetchPosts()

let result = await (user, posts)
```

* Tasks start together
* Suspend independently
* Still structured and safe

---

## Async Error Handling

```swift
func fetchUser() async throws -> User
```

Calling it:

```swift
do {
    let user = try await fetchUser()
} catch {
    // Handle error
}
```

* Linear
* Local
* Predictable

---

## Why `await` Is Mandatory

Swift requires `await` because:

* Suspension changes execution order
* Code after `await` may run much later
* Shared state may change

This makes async behavior explicit.

---

## Threading Reality

```swift
print(Thread.current)
let user = await fetchUser()
print(Thread.current)
```

* Thread before and after `await` may differ
* Thread identity must not be relied upon

---

## Async Does NOT Mean Background

```swift
@MainActor
func loadData() async {
    let data = await fetchData()
    updateUI(data)
}
```

* Runs on the main actor
* Safely suspends
* UI remains responsive

---

## Rules to Remember

1. `async` functions can suspend
2. `await` marks suspension points
3. Suspension does not block threads
4. Execution may resume on a different thread
5. Async code is sequential by default
6. Concurrency must be explicit

---

## Common Mistakes

* Assuming async means background execution
* Forgetting `await`
* Relying on thread identity
* Expecting parallel execution by default
* Updating UI without `@MainActor`

---

## Mental Model

> **Async / await is about suspending functions, not threads.**

---


