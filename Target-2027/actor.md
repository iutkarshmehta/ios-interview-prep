# Actors in Swift Concurrency

## 1. What is Actors?

An `actor` in Swift is a nominal reference type that protects its internal mutable state by ensuring mutually exclusive access. It isolates its state from the rest of the program, meaning only one task can access or modify its mutable properties at any given time.

At a Staff engineering level, you should not view an actor simply as a "class with a lock." Instead, an actor is a fundamental concurrency primitive integrated directly into the Swift compiler and runtime. It defines an **isolation domain**. When code outside this domain wishes to interact with the actor's state, it must do so asynchronously via message passing (using `await`), yielding control to the Swift concurrency cooperative thread pool until the actor's executor is free to process the message.

## 2. Why do we need it?

We need actors to safely share mutable state across concurrent execution contexts without relying on manual, error-prone synchronization mechanisms.

As iOS applications become highly concurrent, managing shared state (like an image cache, a session manager, or a database coordinator) becomes a minefield for data races. Before Swift Concurrency, developers had to rely strictly on developer discipline to remember to lock a mutex or dispatch to a specific serial queue before accessing a variable. Developer discipline does not scale in large teams or complex codebases.

## 3. What problem does it solve?

Actors solve **data races at compile time**.

A data race occurs when two concurrent threads access the same memory location, at least one is a write, and there is no synchronization.

The problem with traditional synchronization (locks, queues) is that it is a *runtime* mitigation. If you forget to lock before writing, the compiler says nothing; the app simply crashes or corrupts data in production. Actors solve this by elevating synchronization to a *type-system level* feature. The compiler strictly prohibits synchronous cross-actor access to mutable state, shifting concurrency bugs from runtime panics to compile-time errors.

## 4. What existed before it?

Before actors, iOS engineers used:

1. **`NSLock` / `os_unfair_lock**`: Synchronous locking mechanisms.
* *Limitations:* Blocking the thread leads to context-switching overhead and potential thread starvation. Highly prone to deadlocks if locks are acquired in the wrong order. Easy to forget to unlock (though `defer` helped).


2. **Serial `DispatchQueue**`: Using GCD to serialize access.
* *Limitations:* Easy to accidentally capture variables unsafely. `DispatchQueue.sync` can cause instant deadlocks if called from the same queue.


3. **Dispatch Barriers (`.barrier`)**: Concurrent reads, exclusive writes on a concurrent queue.
* *Limitations:* Extremely verbose, hard to read, and still purely reliant on the developer remembering to use the barrier flag.



Without actors, developers typically wrapped their classes in private queues and exposed asynchronous completion handlers or synchronous getters. The danger was always human error: leaking unsynchronized state or causing priority inversions and deadlocks.

## 5. Core mental model

The Staff-level mental model for an actor is a **Reentrant Mailbox on an Island**.

* **The Island:** The actor's state is completely isolated. Nothing gets in or out without passing through the border.
* **The Mailbox:** External callers cannot manipulate the state directly. They must put a message in the mailbox (by calling an `await` method).
* **The Executor:** The island has one worker (the serial executor). It takes one message from the mailbox, processes it completely, and then takes the next.
* **Reentrancy:** If the worker has to wait for something (an internal `await`, like a network call), *it does not sit idle blocking the mailbox*. It sets the current task aside and processes the *next* message in the mailbox. **This prevents deadlocks but allows state interleaving.**

## 6. How does it work?

### Compile-time behavior

The Swift compiler statically analyzes code for **actor isolation**. If a caller is outside the actor's isolation domain, the compiler forces the caller to use `await` to read or write. Furthermore, the compiler enforces `Sendable` checking. Data passed into or out of an actor *must* conform to `Sendable` (meaning it is safe to share concurrently), preventing the leakage of mutable reference types across actor boundaries.

### Runtime behavior

Actors do not map 1:1 to OS threads. They use a **Serial Executor**. When an actor method is called, the task is enqueued on the actor's executor. The Swift cooperative thread pool (which matches the number of CPU cores) schedules these tasks. Because the thread pool is limited, actors use lightweight continuations instead of blocking threads.

### Memory & Dispatch behavior

Inside the actor, methods are dispatched statically or via the vtable, just like a class, because isolation is guaranteed. Cross-actor calls are transformed by the compiler into asynchronous task submissions to the actor's executor. Actors use ARC (Automatic Reference Counting) just like classes.

## 7. Key concepts and terminology

* **Isolation Domain:** The boundary protecting the actor's state. Code is either *actor-isolated* (inside) or *non-isolated* (outside).
* **Cross-actor reference:** Accessing an actor from outside its isolation domain.
* **`Sendable`:** A protocol indicating a type is safe to pass across concurrency domains (e.g., value types, actors, or final classes with immutable state).
* **Reentrancy:** The ability for an actor to suspend a task at an `await` point and begin executing a new, different task before the first one resumes.
* **Executor:** The service that accepts jobs and executes them serially for the actor.
* `nonisolated`: A keyword to opt a specific method/property out of actor isolation (e.g., for conforming to protocols, provided it doesn't touch mutable isolated state).

## 8. Basic implementation

```swift
// A minimal actor protecting an integer counter.
actor HitCounter {
    // Isolated mutable state
    private(set) var count = 0 
    
    // Isolated method
    func increment() {
        // No locks needed; mutually exclusive access is guaranteed
        count += 1
    }
}

// Usage (from outside the actor)
Task {
    let counter = HitCounter()
    // 'await' is required because we are crossing the isolation boundary
    await counter.increment()
    print(await counter.count) 
}

```

* *What it solves:* Concurrent increments won't overwrite each other.
* *What happens if removed:* `count` would suffer from a data race if called from multiple threads.

## 9. Production implementation

A production-quality authenticated network session manager.

```swift
/// An actor managing the refresh of an OAuth authentication token.
actor SessionManager {
    private var currentToken: AuthToken?
    private var activeRefreshTask: Task<AuthToken, Error>?
    private let networkClient: NetworkClient // Must be Sendable
    
    init(networkClient: NetworkClient) {
        self.networkClient = networkClient
    }
    
    /// Retrieves a valid token, refreshing if necessary.
    func getValidToken() async throws -> AuthToken {
        // 1. Return current if valid
        if let token = currentToken, token.isValid {
            return token
        }
        
        // 2. Coalesce duplicate requests. 
        // If a refresh is already in flight, await its result instead of triggering a new one.
        if let existingTask = activeRefreshTask {
            return try await existingTask.value
        }
        
        // 3. Create a new task to refresh the token
        let task = Task {
            let newToken = try await networkClient.refreshToken()
            return newToken
        }
        
        self.activeRefreshTask = task
        
        // 4. Await the result of our own task
        do {
            let newToken = try await task.value
            self.currentToken = newToken
            self.activeRefreshTask = nil
            return newToken
        } catch {
            self.activeRefreshTask = nil
            throw error
        }
    }
}

```

* *What it does:* Prevents the "thundering herd" problem where 50 concurrent network requests all trigger a token refresh simultaneously.
* *Why it's needed:* `activeRefreshTask` acts as a request coalescer.
* *Staff Nuance:* Notice we assign the `Task` to state *before* we `await`. Because actors are reentrant, if we `await networkClient.refreshToken()` directly, another request could enter this method while the first is suspended, causing duplicate network calls.

## 10. Real-world iOS use cases

1. **`@MainActor` for ViewModels:** In SwiftUI/UIKit, view state must be mutated on the main thread. `@MainActor` is a global actor that ensures all state mutations happen on the Main Dispatch Queue.
2. **Persistence Coordinators:** An actor wrapping a Core Data `NSManagedObjectContext` (specifically a background context) or SwiftData `ModelActor` to serialize database writes.
3. **In-Memory Caches:** Image caches or data caches where multiple screens might request the same resource concurrently.
4. **Analytics Batching:** An actor that accumulates analytics events and periodically flushes them to the network serially.

## 11. Alternatives

1. **`os_unfair_lock`**
* *What it does:* An OS-level, low-level lock.
* *Advantage:* Extremely fast when uncontended. Completely synchronous (no `await` viral spread).
* *Disadvantage:* Blocks the thread. Can cause deadlocks. Requires manual management.
* *When to use:* High-performance, synchronous critical sections (e.g., inside a logging framework or heavy math computations) where crossing an actor boundary (`await`) is too slow.


2. **DispatchQueue (Serial)**
* *Advantage:* Familiarity. Supports synchronous blocking (`queue.sync`).
* *Disadvantage:* Thread-hopping overhead. Can cause instant deadlocks.
* *When to use:* Interacting with legacy C-APIs or Objective-C code that expects queues.


3. **Value Semantics (Structs + Copy-on-Write)**
* *Advantage:* Thread-safe by default because every thread gets its own copy. No synchronization overhead.
* *Disadvantage:* Cannot be used to share *global* state (like a single network session or a single cache).



## 12. Trade-offs

* **Safety vs. Synchrony:** Actors force you to use `await` from the outside. This "colors" your functions, forcing the caller to be `async`. You cannot synchronously read a value from an actor from non-isolated code.
* **Reentrancy vs. Atomicity:** Because actors are reentrant, they prevent deadlocks but sacrifice transaction atomicity across `await` points. You cannot assume the state of the actor after an `await` is the same as it was before the `await`.
* **Cooperative Pool vs. Thread Priority:** Heavy synchronous work inside an actor blocks the executor. Because the cooperative pool has a fixed thread count, blocking an actor can starve the entire application of concurrency.

## 13. Common mistakes

### 1. The Reentrancy Trap (Senior/Staff Level Mistake)

* *Mistake:* Assuming state remains unchanged across an `await`.
```swift
actor BankAccount {
    var balance: Decimal = 100

    func withdraw(amount: Decimal) async {
        guard balance >= amount else { return }
        // SUSPENSION POINT: Another task can enter here!
        await networkServer.verifyTransaction() 
        // DANGER: `balance` might have changed while we were suspended.
        balance -= amount 
    }
}

```


* *Why it happens:* Treating `actor` like a synchronous `NSLock`.
* *Correction:* Always re-check state after a suspension point, or perform mutations *before* suspending.

### 2. Passing Non-Sendable Types (Intermediate)

* *Mistake:* Passing an NSMutableDictionary or an unprotected class into an actor.
* *Why it's wrong:* The caller holds a reference, the actor holds a reference. The caller can mutate it synchronously while the actor mutates it, causing a data race.
* *Correction:* Swift 6 strict concurrency checks this by default. Ensure all cross-actor communication uses `Sendable` types.

### 3. Starving the Thread Pool

* *Mistake:* Running a heavy `UIImage` filter synchronously inside an actor.
* *Correction:* Move heavy CPU work to a detached task or a separate actor, or use non-isolated functions.

## 14. Edge cases

* **Deadlocks:** While actors prevent traditional locking deadlocks via reentrancy, you can still create logical deadlocks if Actor A waits for a `Task` that waits for Actor A to finish a state change that is currently suspended.
* **Global Actors:** `@MainActor` is a unique actor tied directly to the main thread. It bridges the actor model with the legacy UI RunLoop.
* **Initialization:** During `init`, an actor is not fully isolated until all properties are initialized. You cannot call isolated methods inside `init` until initialization is complete.
* **Objective-C Interoperability:** Swift actors are not exposed to Objective-C. You must provide `nonisolated` `@objc` wrappers or use `Task` bridging.

## 15. Performance

* **Context Switching:** Actor hopping does not necessarily mean an OS thread context switch. If the cooperative thread pool determines it's efficient, it will execute the continuation on the same thread.
* **Synchronization Overhead:** Dispatching to an actor is generally faster than a GCD queue hop but slower than an uncontended `os_unfair_lock`.
* **When to avoid:** Do not use an actor for a simple struct that needs to be accessed 10,000 times per frame in a game loop. The `await` suspension overhead will destroy frame rates. Use `os_unfair_lock` or atomic variables for ultra-high-frequency synchronous state.

## 16. Architecture

Actors naturally fit into the **Service** or **Repository** layer of Clean Architecture / MVVM.

* **Do use actors for:** Singletons that manage shared resources (e.g., `DatabaseManager`, `NetworkSessionCoordinator`, `AudioPlaybackEngine`).
* **Do not use actors for:** Pure data models (use `structs`), pure stateless utility functions (use enums with static functions), or ViewModels (unless they are bound to `@MainActor`).
* **Modularization:** When building SDKs, keep actor boundaries internal. Exposing actors forces consumers to adopt Swift Concurrency. Often, SDKs expose an `async` interface but hide the `actor` internally.

## 17. Testing

Testing actors requires asynchronous test contexts.

```swift
final class SessionManagerTests: XCTestCase {
    func testTokenCoalescing() async throws {
        let mockNetwork = MockNetworkClient()
        let actor = SessionManager(networkClient: mockNetwork)
        
        // Spawn multiple concurrent tasks
        async let token1 = actor.getValidToken()
        async let token2 = actor.getValidToken()
        
        let results = try await [token1, token2]
        
        // Assert that the network client was only hit once
        XCTAssertEqual(mockNetwork.refreshCallCount, 1)
    }
}

```

* *Determinism:* Concurrency makes tests flaky. Ensure your mocks (like `MockNetworkClient`) use `continuation`s to precisely control *when* suspension points resolve, allowing you to explicitly test reentrancy states.

## 18. Migration / legacy considerations

Migrating a legacy UIKit app heavily reliant on GCD:

1. **Incremental Adoption:** Use `@preconcurrency import` to suppress warnings for legacy modules.
2. **`withCheckedContinuation`:** Bridge delegate callbacks or completion handlers into `async` contexts before moving the logic into an actor.
3. **Global Actors:** Apply `@MainActor` to older ViewControllers and ViewModels to safely integrate them with new actor-based services.
4. *Staff Insight:* Do not blindly replace every `class` + `DispatchQueue` with an `actor`. Re-evaluate if the state actually *needs* to be shared. If you can refactor to value types, do that first.

## 19. Staff-level design scenarios

**Scenario:** *You are working on a healthcare iOS app with 5 million users. The app syncs 50,000 core data records continuously in the background. The UI must remain perfectly responsive. How do you design the sync engine using Swift Concurrency?*

* **Design:**
1. Create a `SyncCoordinator` (Actor) to serialize sync batches.
2. Use a `ModelActor` (SwiftData) or a custom background context (Core Data) running on a custom executor to ensure data writes do not block the default cooperative pool.
3. Pass only `Sendable` structs (DTOs) from the networking layer to the `SyncCoordinator`.
4. Yield control. Since parsing 50k records is CPU-intensive, use `await Task.yield()` inside the parsing loop to voluntarily yield the thread back to the cooperative pool, preventing starvation of other concurrent tasks.


* **Trade-offs:** Yielding increases total sync time but guarantees UI and other background tasks (like fetching images) aren't starved.
* **Failure modes:** If network fails, the actor isolates the retry logic. Reentrancy ensures that while waiting for exponential backoff, the actor can still respond to a "cancel sync" message from the UI.

## 20. Interview questions

* **LEVEL 1 — Fundamentals:** What is the difference between a `class` and an `actor`?
* *Answer:* Both are reference types, but an actor isolates its state, guaranteeing mutually exclusive access.


* **LEVEL 2 — Intermediate:** Why do you have to use `await` when calling an actor's method?
* *Answer:* Because the call crosses the isolation boundary. It might have to suspend if the actor's executor is currently busy processing another message.


* **LEVEL 3 — Senior:** What does `@MainActor` actually do under the hood?
* *Answer:* It is a global actor that uses a custom executor bound to the main thread's runloop, ensuring all code isolated to it executes serially on the main thread.


* **LEVEL 4 — Staff:** Explain actor reentrancy and the bugs it can cause.
* *Answer:* Actors can suspend at an `await`, freeing the executor to process other messages. This prevents deadlocks but means actor state can change across the `await`. If an engineer assumes atomicity across an `await`, they will corrupt the state.


* **LEVEL 5 — Expert:** How would you integrate an existing heavily threaded C++ library with Swift Concurrency?
* *Answer:* Wrap it in an actor, but carefully manage the threading. If the C++ library blocks, it will block the Swift cooperative pool. I would use custom executors (introduced in Swift 5.9) to isolate the C++ blocking calls to a dedicated thread pool, freeing the main Swift pool.



## 21. Deep follow-up questions

* *Interviewer:* "You said actors prevent deadlocks via reentrancy. Can you ever deadlock an actor?"
* *Staff Answer:* Yes. If you mix actors with traditional locking (e.g., using `NSLock` inside an actor and suspending), or if two actors rely on `async let` tasks that mutually depend on each other's state to complete, you can create a structural or logical deadlock, even if the executor itself isn't blocked.


* *Interviewer:* "Why can't we just use a serial `DispatchQueue` instead?"
* *Staff Answer:* Queues lack compile-time safety. You can easily capture non-thread-safe references in a queue block. Furthermore, GCD relies on thread-hopping, which is computationally expensive. Swift Concurrency's cooperative pool uses continuations, minimizing actual OS thread context switches.


* *Interviewer:* "What happens if a network call inside an actor takes 30 seconds?"
* *Staff Answer:* The task suspends. Because the actor is reentrant, the actor's executor is freed up to handle other method calls immediately. The thread is not blocked.



## 22. Common interview traps

* **The "Thread" Trap:** Saying "an actor runs on its own thread." *Wrong.* Actors run on executors, which are scheduled on a shared cooperative pool of threads. They are not bound to a specific thread (except `@MainActor`).
* **The "Data Race" Trap:** Assuming actors solve *all* race conditions. Actors prevent *data races* (memory corruption). They do not prevent *race conditions* (logical ordering bugs).
* **The "Sendable" Trap:** Forgetting that returning a `class` instance from an actor defeats the purpose of the actor, as the caller now has an unprotected reference. The compiler warns/errors on this in Swift 6, but interviewers will test if you understand *why*.

## 23. Staff Engineer mental model

> "Actors are compile-time enforced, reentrant synchronization boundaries. I use them exclusively for shared mutable reference state. I design my actors keeping reentrancy top of mind, ensuring state invariants are valid before every `await`. I prefer value types and purely functional transformations whenever possible, utilizing actors only at the necessary synchronization points (like a database or network throttle) to avoid coloring my entire codebase async and incurring suspension overhead."

## 24. Quick revision sheet

```markdown
# Actor Quick Revision
- **Type:** Reference Type (AnyObject)
- **Problem solved:** Data Races (Compile-time)
- **State:** Isolated (requires `await` to access from outside)
- **Executor:** Serial by default, scheduled on cooperative pool.
- **Reentrancy:** YES. State can mutate during `await`. Always validate state post-await.
- **Sendable:** Enforced. Types crossing boundary must be thread-safe.
- **Global Actor:** `@MainActor` (bound to Main Thread).
- **Overhead:** Suspension/Continuation cost. Slower than `os_unfair_lock`, safer than GCD.
- **Usage:** Shared resources (Caches, Managers, Coordinators).

```

## 25. Official sources

Information synthesized strictly from the following official Apple and Swift.org architectural documents and proposals:

1. **** Swift Evolution Proposal SE-0306: Actors. (Original foundational design, reentrancy, isolation).
2. **** WWDC21 Session 10254: Swift concurrency: Behind the scenes. (Cooperative thread pool, executors, continuations).
3. **** Swift Evolution Proposal SE-0302: `Sendable` and `@Sendable` closures. (Cross-actor boundary enforcement).
4. **** Swift Evolution Proposal SE-0316: Global Actors. (`@MainActor` and UI isolation).
5. **** Swift Evolution Proposal SE-0392: Custom Actor Executors. (Swift 5.9 custom runtime behaviors).
6. **** Swift Evolution Proposal SE-0337: Strict Concurrency Checking. (Migration, preconcurrency).