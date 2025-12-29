### Sendable Protocol

### What problem Sendable solves

* Swift Concurrency allows data to move between concurrent tasks
* Moving data across concurrency domains can cause data races
* `Sendable` tells the compiler a type is **safe to transfer across task/actor boundaries**

### What Sendable means

* A type marked `Sendable` guarantees:

  * No shared mutable state
  * Or internal synchronization that makes it safe
* The compiler uses `Sendable` to **prevent unsafe data sharing at compile time**

### Where Sendable is checked

* Passing data between tasks
* Passing data into `Task {}` or `Task.detached {}`
* Crossing actor boundaries
* Returning values from async functions

---

### Value Types and Sendable

### Why value types are usually Sendable

* Value types are copied on transfer
* No shared memory after the copy
* This naturally prevents data races

### Automatically Sendable value types

* `struct`, `enum`, `tuple` whose stored properties are all `Sendable`
* Common standard types:

  * `Int`, `String`, `Bool`
  * `Array`, `Dictionary` (if elements are Sendable)

```swift
struct User: Sendable {
    let id: Int
    let name: String
}
```

### When value types are NOT Sendable

* If they contain:

  * Non-Sendable reference types
  * Unsafe pointers
  * Mutable shared references

```swift
struct Cache {
    var store: NSMutableDictionary // ❌ not Sendable
}
```

---

### Reference Types and Sendable

### Why reference types are unsafe by default

* Reference types share memory
* Multiple tasks can access the same instance concurrently
* This leads to data races without protection

### Default behavior

* `class` types are **not Sendable**
* Compiler prevents passing them across concurrency boundaries

```swift
class Session {
    var token: String
}
```

Passing `Session` across tasks → compile-time error

---

### How reference types can become Sendable

### 1. Immutable reference types

* All stored properties are constants (`let`)
* No mutation after initialization

```swift
final class Config: Sendable {
    let baseURL: URL
}
```

### 2. Internally synchronized reference types

* Uses locks, actors, or other synchronization
* You guarantee thread safety manually

---

### @unchecked Sendable

### What @unchecked Sendable means

* You are telling the compiler:

  * “Trust me, this type is safe”
* Compiler **stops enforcing safety checks**
* Responsibility shifts entirely to you

```swift
final class LegacyManager: @unchecked Sendable {
    private let lock = NSLock()
    private var state: Int = 0
}
```

---

### When to use @unchecked Sendable

* Wrapping legacy thread-safe code
* Reference types with proven synchronization
* Performance-critical paths where checks are too strict

### When NOT to use it

* As a shortcut to silence compiler errors
* When safety is unclear
* In new code without strong guarantees

---

### Relationship Between Actors and Sendable

* Actors protect mutable state → actor references are Sendable
* You can safely pass actor instances across tasks
* Actor isolation prevents races automatically

---

### Staff Engineer Rules to Remember

* Prefer value types → automatic Sendable
* Prefer actors over locks
* Avoid `@unchecked Sendable` unless unavoidable
* Treat compiler warnings as design feedback

---

### Final Mental Model

* `Sendable` defines **what data may cross concurrency boundaries**
* Value types are safe by default
* Reference types must prove safety
* `@unchecked Sendable` trades compiler safety for responsibility
