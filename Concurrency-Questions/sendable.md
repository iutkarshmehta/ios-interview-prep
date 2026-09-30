

---

# **Sendable Types – Different Perspectives**

## **1️⃣ Conceptual Perspective**

* **What it really means:**
  A Sendable type is a type that **can be safely shared between concurrent tasks or actors** without causing **data races**.
* **Mental image:**

  > Think of a Sendable type as a **“sealed envelope”** — you can hand it to someone else (another task/actor) without worrying that someone else is changing the contents at the same time.

---

## **2️⃣ Safety Perspective**

* **Sendable ensures:**

  * Immutable values are safe.
  * No two tasks can **mutate the same instance** at the same time.
  * Compiler checks enforce this at **compile-time**.

* **Non-Sendable types:**

  * Mutable classes (shared state)
  * Collections containing non-Sendable items
  * Any type where concurrency could break safety

---

## **3️⃣ Type Perspective**

### ✅ Sendable by default

| Type                           | Reason                      |
| ------------------------------ | --------------------------- |
| `Int`, `Bool`, `Double`        | Primitive, immutable        |
| `String`                       | Value type, immutable       |
| Immutable `struct`             | All properties are Sendable |
| `Array<T>` where `T: Sendable` | All elements are safe       |

### ❌ Not Sendable by default

| Type                      | Reason                                   |
| ------------------------- | ---------------------------------------- |
| `class Counter`           | Mutable, shared reference                |
| `NSMutableArray`          | Mutable, unsafe for concurrent access    |
| `Dictionary<String, Any>` | Any non-Sendable element makes it unsafe |

---

## **4️⃣ Concurrency Perspective**

* **Why Sendable matters:**
  Swift concurrency (async/await, tasks, actors) moves data between threads/tasks.

  * Passing a non-Sendable type could cause **race conditions**.
  * Sendable types **guarantee thread-safe transfer**.

---

## **5️⃣ Practical Perspective**

* **Immutable structs:** automatically Sendable
* **Mutable classes:** use `@unchecked Sendable` only if you can **guarantee safety manually**

```swift id="sendable5"
struct User: Sendable {
    let name: String
    let age: Int
}

class Counter {
    var count = 0
}
extension Counter: @unchecked Sendable {} // developer promises safety
```

* **Tip:** prefer structs and immutability — easiest way to make types Sendable.

---

## **6️⃣ Interview Way to Explain (Different Angle)**

> “Sendable types in Swift concurrency are types that can safely be passed across tasks or actors without risking data races.
> You can think of them as **sealed, immutable envelopes**. Primitive types like `Int` and `String`, and immutable structs are Sendable by default. Classes with mutable state are not Sendable by default because multiple tasks could change the same object at the same time. Swift enforces Sendable types at compile time to ensure concurrency safety.”

---

If you want, I can **now integrate Sendable types into our full Swift concurrency Markdown sheet** so you have **Main Thread, Main Actor, Executors, Concurrency Abstraction, and Sendable types all in one single-page revision**.

Do you want me to do that?
