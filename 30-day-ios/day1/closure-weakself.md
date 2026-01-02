### What is a Closure (Interview Definition)

A closure is a block of executable code that can be stored, passed around, and executed later.
Closures can **capture and store references to variables and constants from their surrounding context**.

Key points:

* Functions are a special case of closures
* Closures are **reference types**
* Widely used for callbacks, async code, and event handling

---

### What Does “Capture Values From Surrounding Context” Mean

Capture means:
A closure can **remember and use variables defined outside it**, even after the original scope has finished executing.

Example:

```swift
func makeCounter() -> () -> Int {
    var count = 0

    let closure = {
        count += 1
        return count
    }

    return closure
}
```

```swift
let counter = makeCounter()
counter() // 1
counter() // 2
```

Explanation:

* `makeCounter()` has already finished
* `count` should normally be destroyed
* The closure **captures `count`**, keeping it alive in memory

---

### How Closures Capture Value Types

```swift
var x = 10
let closure = {
    print(x)
}

x = 20
closure() // prints 20
```

Important detail:

* Swift captures a **reference to the variable**, not a snapshot
* Value is stored on heap if the closure needs it

---

### How Closures Capture Reference Types

```swift
class User {
    var name = "A"
}

let user = User()

let closure = {
    print(user.name)
}
```

Explanation:

* Closure captures `user` **strongly**
* Retain count of `user` increases

This is where memory issues begin in iOS apps.

---

### Why Closures Cause Retain Cycles

```swift
class ViewController {
    var completion: (() -> Void)?

    func setup() {
        completion = {
            self.loadData()
        }
    }

    func loadData() {}
}
```

What happens:

* ViewController strongly holds `completion`
* Closure strongly captures `self`
* Neither can be released

Result:
❌ Retain cycle → memory leak

---

### What `[weak self]` Does

```swift
completion = { [weak self] in
    self?.loadData()
}
```

Effect:

* `self` is captured **weakly**
* Retain count is **not increased**
* `self` becomes optional
* If VC deallocates, `self` becomes `nil`

---

### Why `self` Becomes Optional With Weak

Weak references:

* Do not keep objects alive
* Are automatically set to `nil` on deallocation

Swift forces optional handling to prevent crashes.

---

### Common Safe Patterns With `[weak self]`

Using guard:

```swift
completion = { [weak self] in
    guard let self else { return }
    self.loadData()
}
```

Short form:

```swift
completion = { [weak self] in
    self?.loadData()
}
```

---

### Escaping Closures and Memory Risk

```swift
func fetchData(completion: @escaping () -> Void) {
    DispatchQueue.global().async {
        completion()
    }
}
```

Why `@escaping`:

* Closure outlives the function scope
* Stored or executed later

Important:

* Escaping closures are **more likely** to cause retain cycles

---

### When Retain Cycles Commonly Happen in iOS

* Closures stored as properties
* GCD async blocks
* Network callbacks
* Timers
* Animations

---

### Cross-Questions Interviewers Ask

**Why not always use `[weak self]`?**

* Optional handling
* Some logic must not run if `self` is gone
* `unowned` expresses guaranteed lifetime

---

**Can non-escaping closures cause retain cycles?**

* Usually no
* Because closure doesn’t outlive the scope

---

**Is `[weak self]` required in async/await?**

* Yes, when task lifetime can exceed object lifetime
* Especially in ViewControllers

---

### One-Line Interview Summary

Closures extend object lifetimes.
ARC manages reference counting but **cannot break retain cycles** — developers must.

---

### The Question Interviewers Ask

**“Why not always use `[weak self]` in closures?”**

At first glance, it feels like:

> weak = safe → so use it everywhere

But that is **not correct**, and interviewers want to see if you understand *why*.

---

### First: What `[weak self]` Really Means

When you write:

```swift
{ [weak self] in
    self?.doSomething()
}
```

You are saying:

> “This closure does NOT own `self`.
> If `self` is gone, this code should silently do nothing.”

That is a **semantic decision**, not just a memory fix.

---

### Problem 1: `[weak self]` Can Silently Skip Important Logic

Consider this example:

```swift
class DataManager {
    func save() {
        print("Saved")
    }

    func performSave() {
        DispatchQueue.global().async { [weak self] in
            self?.save()
        }
    }
}
```

If `DataManager` deallocates before the async block runs:

* `self` becomes `nil`
* `save()` is **never called**
* No crash
* No warning
* Logic just disappears

This is **dangerous** if the work is important.

---

### Interview Insight #1

Using `[weak self]` means:

> “It is acceptable for this code to NOT execute.”

If that assumption is wrong → **bug**.

---

### Problem 2: Weak Makes `self` Optional (Noise + Complexity)

With `[weak self]`, you must handle optional `self`:

```swift
self?.doSomething()
```

or

```swift
guard let self else { return }
```

This adds:

* extra branching
* less readable code
* cognitive overhead

If `self` is **guaranteed to exist**, this is unnecessary.

---

### Problem 3: Overusing Weak Hides Ownership Problems

Bad mindset:

> “I’ll just add `[weak self]` everywhere to be safe.”

This hides:

* incorrect object ownership
* bad architecture
* unclear lifetimes

Senior engineers prefer:

> clear ownership + correct lifetimes
> not blanket weak references

---

### When `[weak self]` IS the Right Choice (Very Important)

Use `[weak self]` when **ALL are true**:

* The closure may outlive `self`
* The closure is asynchronous or stored
* It is acceptable to skip execution if `self` is gone

Typical cases:

* Network callbacks
* GCD async blocks
* Timers
* Animations
* Long-running tasks in ViewControllers

Example:

```swift
Task { [weak self] in
    await self?.loadData()
}
```

Perfect use of `weak`.

---

### When `[weak self]` Is NOT the Right Choice

Case 1: Closure lifetime ≤ object lifetime

```swift
class User {
    let name: String

    lazy var uppercaseName: String = { [unowned self] in
        self.name.uppercased()
    }()
}
```

Here:

* Closure cannot outlive `self`
* Using `weak` would be unnecessary
* `unowned` expresses correct ownership

---

### Weak vs Unowned (Conceptual Difference)

* `weak` → “self may disappear”
* `unowned` → “self will NOT disappear”

Using `unowned` is **stronger documentation** of intent.

---

### Interview Insight #2 (Very Important)

Interviewers are testing:

* Do you understand **lifetime guarantees**
* Do you know **when logic must run**
* Do you reason beyond memory leaks

Not:

* whether you memorized `[weak self]`

---

### The Correct Mental Model (Remember This)

> Use `[weak self]` **only when it is okay for the code to not run**
> Use `unowned` when lifetime is guaranteed
> Use strong `self` when execution must happen

---

### How to Say This in an Interview (Perfect Answer)

> “I don’t always use `[weak self]`.
> I use it when the closure can outlive the object and it’s acceptable for the logic to be skipped if the object is deallocated.
> If the lifetime is guaranteed, I prefer `unowned` or strong references for clarity.”


