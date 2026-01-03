### Value Types vs Reference Types — Structs, Classes, ARC, and Copy-on-Write

*(Staff Engineer + Tutor Mode)*

---

### First Principles: Why This Distinction Exists

Swift did **not** accidentally have both `struct` and `class`.
This split exists because **not all data should behave the same way**.

At a very deep level, the question is:

> When I pass data around, do I want to **share identity** or **copy meaning**?

Swift encodes this decision directly into the type system.

---

### Concept: Value Types vs Reference Types

### Value Types (Struct, Enum, Tuple)

A value type represents **a value, not an identity**.

Core rule:

> Assigning or passing a value type creates a **new independent copy**.

```swift
struct Point {
    var x: Int
    var y: Int
}

var p1 = Point(x: 0, y: 0)
var p2 = p1
p2.x = 10
```

`p1` is unchanged.

Mental model:

* Like `Int`, `Bool`, `String`
* “What matters is the value, not who owns it”

---

### Reference Types (Class)

A reference type represents **a shared identity**.

Core rule:

> Assigning or passing a class copies the **reference**, not the object.

```swift
class Counter {
    var count = 0
}

let c1 = Counter()
let c2 = c1
c2.count = 10
```

`c1.count` is now `10`.

Mental model:

* Multiple references → one shared instance
* Mutations are visible everywhere

---

### Why Swift Defaults to Structs

Swift designers made a strong choice:

> **Use value semantics by default.**

Reasons:

* Predictability
* Thread safety
* Easier reasoning
* Fewer hidden side effects

Most Swift standard library types are value types:

* `Array`
* `Dictionary`
* `Set`
* `String`

This is intentional.

---

### Mechanics: What Actually Happens in Memory

### Structs (Value Types)

* Stored **inline** or on the stack (conceptually)
* Copied on assignment (logically)
* No reference counting
* No identity

Important:

> “Copied” does not always mean “physically duplicated immediately”
> (we’ll come back to this with Copy-on-Write)

---

### Classes (Reference Types)

* Stored on the heap
* Managed by **ARC**
* Passed by reference
* Have identity (`===`)

```swift
if obj1 === obj2 {
    // same instance
}
```

This check does **not exist** for structs.

---

### ARC and Why It Only Applies to Classes

ARC = Automatic Reference Counting

ARC tracks:

* How many **strong references** point to a class instance
* Deallocates when count reaches zero

Structs:

* Do not use ARC
* Have no reference count
* Cannot participate in retain cycles

This is a **huge advantage** of value types.

---

### When You Must Use a Class

Use a class when **identity matters**.

Typical cases:

* View controllers
* Views
* Coordinators
* Shared mutable state
* Delegate patterns
* Objects with lifecycle (`deinit`)

Rule of thumb:

> If two variables must always reflect the same state → class.

---

### When You Should Prefer a Struct

Use a struct when:

* It models data
* You want immutability by default
* You want thread safety
* You want predictable behavior

Examples:

* Models
* ViewModels
* Configuration objects
* DTOs
* Domain entities

Interview signal:

> “I default to structs unless identity is required.”

That’s a **strong answer**.

---

### Copy-on-Write (CoW): The Best of Both Worlds

At first glance, value types seem expensive:

> “Are arrays copied every time?”

No.

Swift uses **Copy-on-Write**.

---

### Concept: What Copy-on-Write Means

> Copy only when mutation happens and sharing is no longer safe.

Example:

```swift
var a = [1, 2, 3]
var b = a   // no copy yet

b.append(4) // copy happens here
```

Until mutation:

* `a` and `b` share the same storage
* Read-only operations are cheap

On mutation:

* Storage is copied
* Value semantics are preserved

This gives:

* Performance of references
* Safety of values

---

### How Swift Knows When to Copy

Internally:

* Reference count > 1 → shared storage
* Mutation requested → copy before write

This is why:

* Arrays are fast
* Dictionaries are safe
* Structs scale well

---

### CoW and ARC (Important Connection)

Even though arrays are structs:

* Their **internal storage** is a reference type
* ARC is used internally
* Exposed API remains value-semantic

This is a **brilliant design tradeoff**.

---

### Patterns in Production Code

### Good Pattern

```swift
struct User {
    let id: Int
    let name: String
}
```

Immutable, predictable, safe.

---

### Dangerous Pattern

```swift
class User {
    var name: String
}
```

Used everywhere, mutated everywhere, no ownership clarity.

Interview red flag:

* Using classes for models “because UIKit uses classes”

---

### Structs + Mutability Control

```swift
struct Settings {
    private(set) var isEnabled: Bool
}
```

This allows:

* Controlled mutation
* Clear ownership
* Compiler-enforced safety

---

### Pitfalls to Watch For

* Large structs mutated frequently in tight loops
* Classes used where value semantics are expected
* Accidental shared state via reference types
* Assuming structs are “always cheap” without understanding CoW

---

### Mental Model to Lock In

Ask yourself one question:

> “If I change this value here, should it affect others?”

Yes → class
No → struct

---

### Interview Questions (Conceptual + Tricky)

1. Why did Swift choose value types as the default?
2. What problem does Copy-on-Write solve?
3. Why doesn’t ARC apply to structs?
4. Can value types still use heap memory?
5. When is a class strictly required?
6. How does CoW preserve value semantics?
7. What are the dangers of reference semantics?
8. Why are `Array` and `Dictionary` structs?
9. Can Copy-on-Write ever hurt performance?
10. How would you explain value vs reference semantics to a junior dev?

---

