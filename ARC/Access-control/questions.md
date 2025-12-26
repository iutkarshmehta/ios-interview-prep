Access control is not just about hiding code; it is about **defining your API's surface area** and **enforcing architectural boundaries**. As a Staff Engineer, I use access control to prevent my team (and future consumers of my code) from using components in ways I didn't intend.

Questions:-

---

### **Section 1: The Foundations (Screening)**

**Q1: Rank the five access control levels from most restrictive to least restrictive.**

* **Staff SE Expectation:** You must know the order instantly.
1. `private` (Enclosing declaration/scope)
2. `fileprivate` (Current source file)
3. `internal` (Entire module - Default)
4. `public` (Anyone can import, but cannot subclass/override)
5. `open` (Anyone can import, subclass, and override)



**Q2: What is the default access level in Swift if you don't specify one? Are there any exceptions?**

* **Staff SE Expectation:**
* **Answer:** `internal`.
* **The Trap:** "Are there exceptions?"
* **Exceptions:**
1. **Framework Init:** If you write a `public` struct, the implicit `init` is still `internal`. You must explicitly write `public init`. This is a classic "gotcha" when creating libraries.
2. **Protocol Extensions:** Default to the requirement's level.





---

### **Section 2: Architecture & Library Design**

**Q3: Explain the difference between `public` and `open`. Why did Apple introduce this distinction?**

* **Staff SE Expectation:** This is the most important question for anyone building a framework or SDK.
* **`public`:** Classes can be instantiated and methods called from outside the module, but **cannot** be subclassed or overridden.
* **`open`:** Allows subclassing and overriding outside the module.
* **Why?** It's about **Fragile Base Class** protection. By marking a class `public` (but not `open`), you promise consumers they can *use* it, but you reserve the right to change the internal logic without breaking their subclasses. It locks down inheritance to ensure API stability.



**Q4: I have a `User` class. I want the `id` property to be readable by anyone in the app, but only writable by the `User` class itself. How do I achieve this without writing a custom getter method?**

* **Staff SE Expectation:** I am looking for `private(set)`.
* **Code:** `public private(set) var id: String`
* **Why:** This pattern reduces boilerplate. Writing `private var _id; public var id: String { return _id }` is the "old way" (Java style). Swift offers `private(set)` to handle encapsulation cleanly.



---

### **Section 3: Nuance & Implementation Details**

**Q5: What is the difference between `private` and `fileprivate`? Specifically, how do they behave with Extensions?**

* **Staff SE Expectation:**
* **Pre-Swift 4:** `private` meant *only* inside the class `{}` block.
* **Modern Swift:** `private` is visible to the class **and any extensions of that class** as long as they are in the **same file**.
* **The Usage:** You rarely need `fileprivate` anymore. You mostly use `fileprivate` when you have *two different classes* in the same file and one needs to see the other's details. If it's just one class and its extensions, `private` is sufficient.



**Q6: Unit Testing: My classes are `internal` by default. How can I test them in a separate Test Target without making everything `public`?**

* **Staff SE Expectation:**
* **Answer:** `@testable import MyModule`.
* **Concept:** This compiler attribute opens up `internal` types to the test target as if they were `public`.
* **Limit:** It does **not** expose `private` or `fileprivate` properties. If you need to test a `private` function, your design might be wrong (you should test the public behavior, not private implementation), or the code should be refactored.



---

### **Section 4: Advanced Scenarios**

**Q7: Tuples and Access Levels. If I have a function that returns a Tuple `(InternalType, PrivateType)`, what is the access level of that function?**

* **Staff SE Expectation:**
* **Rule:** The access level of a compound type (like a Tuple or Function) is the **most restrictive** of its components.
* **Answer:** The function must be `private`. It cannot be `internal` or `public` because it exposes a `PrivateType` which the caller wouldn't be allowed to see. The compiler will throw an error if you try to make the function `public`.



**Q8: You are building a UI Framework. You have a `SetupView` protocol that you want your internal views to conform to, but you don't want the consumers of your framework to know this protocol exists. Can your public views conform to this internal protocol?**

* **Staff SE Expectation:**
* **Answer:** Yes, but the conformance must be hidden or the protocol methods must not be exposed as part of the public interface.
* **Swift 5.9+ / Opaque Types:** You typically handle "hidden types" by returning `some Protocol`.
* **Constraint:** A `public` type cannot inherit from an `internal` class. But a `public` type *can* conform to an `internal` protocol, provided the protocol requirements are met internally.



---

### **Summary of Keywords I'm Checking For:**

* Module vs. File
* Subclassing limitations (`open` vs `public`)
* `private(set)` pattern
* `@testable`
* Implicit `init` visibility trap