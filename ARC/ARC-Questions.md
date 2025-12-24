
### **Section 1: The Mechanics of ARC**

**Goal:** Assess understanding of how Swift actually manages memory under the hood vs. Garbage Collection.

**Q1: Can you explain how ARC works at a high level? How is it fundamentally different from a Tracing Garbage Collector (like in Java or Kotlin)?**

* **Staff SE expectation:** I want to hear about *compile-time* vs. *runtime*.
* ARC inserts `retain` and `release` calls at compile time. It is deterministic.
* Garbage Collection (GC) is a runtime process that pauses execution to scan for objects.
* **Key Detail:** ARC does not handle reference cycles automatically; GC usually does.



**Q2: The documentation states that reference counting applies only to instances of classes. Why doesn't it apply to Structs or Enums?**

* **Staff SE expectation:** You should mention *Value Types* vs. *Reference Types*.
* Structs/Enums are passed by value (copied), so there is no single shared instance to "count" references for.
* *Bonus:* Mention that structs *can* incur reference counting overhead if they contain reference types (e.g., a Struct holding a Class instance).



---

### **Section 2: Handling Reference Cycles (Class-to-Class)**

**Goal:** Test the ability to model object relationships and lifecycles correctly.

**Q3: We have a `Person` class and an `Apartment` class. A Person acts as a Tenant. Explain how you would model the relationship to avoid a memory leak. Why did you choose that specific reference type?**

* **Scenario:** `Person` has an `apartment` property. `Apartment` has a `tenant` property.
* **Staff SE expectation:**
* One link must be strong, the other `weak` or `unowned`.
* **Correct Approach:** `Apartment` should likely have `weak var tenant: Person?`.
* **Reasoning:** An apartment can exist empty (without a tenant). The `tenant` has a shorter lifecycle than the `Apartment`.
* **Why not `unowned`?** Because the tenant can be nil. `unowned` is for non-optionals.



**Q4: Deep Dive: When would you ever use `unowned` over `weak`? Give me a concrete architectural example.**

* **Staff SE expectation:** `unowned` is used when the other instance has the **same lifetime or longer**.
* **Example:** A `CreditCard` class and a `Customer` class. A credit card *must* have a customer. It cannot exist in a vacuum.
* `class CreditCard { unowned let customer: Customer ... }`
* *Risk assessment:* Accessing an `unowned` reference after deallocation causes a hard crash (runtime error), whereas `weak` safely resolves to `nil`.



---

### **Section 3: Closures and Capture Lists**

**Goal:** Closures are the #1 source of leaks in Swift apps. This is a critical screening area.

**Q5: Look at this code snippet. Is there a leak? If so, how do we fix it?**

```swift
class HTMLElement {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    // ... init ...
}

```

* **Staff SE expectation:** Yes, there is a leak.
* **The Cycle:** The instance holds the closure (`asHTML`), and the closure captures `self` strongly.
* **The Fix:** Use a capture list: `[unowned self]` or `[weak self]` in the closure.
* **Follow up:** Why is `lazy` required here? (Answer: You cannot access `self` during initialization unless the property is lazy).



**Q6: In a closure capture list, when should you use `[unowned self]` versus `[weak self]`?**

* **Staff SE expectation:**
* **Weak:** If the object (self) might become nil while the closure is waiting to execute or executing (e.g., a network request callback that finishes after the view controller is dismissed).
* **Unowned:** If the closure *cannot* exist longer than the instance (e.g., a lazy property closure like the `HTMLElement` example above, where the closure is owned by `self` and will be destroyed when `self` is destroyed).



---

### **Section 4: Advanced/System Design**

**Goal:** Understanding performance and "Best Practices" at scale.

**Q7: ARC is fast, but it isn't free. What are the performance costs associated with ARC, and how might that influence your decision to use Classes vs. Structs in a high-performance rendering loop?**

* **Staff SE expectation:**
* **Atomicity:** ARC operations (retain/release) must be atomic to be thread-safe. Atomic operations are expensive (overhead).
* **Cache Misses:** Indirect access to classes can cause cache misses.
* **Design Choice:** In a tight loop (e.g., processing thousands of pixels or JSON objects), prefer **Structs**. Since they don't use ARC (mostly), you save the retain/release overhead.



**Q8: Explain "Side Table" memory in the context of `weak` references. What happens to the memory of an object when only `weak` references point to it?**

* **Staff SE expectation:** This is a "Bonus/Expert" level question.
* When the strong count hits zero, the object's *data* is deinitialized.
* However, the memory might not be fully freed immediately if `weak` references exist. Swift uses a "Side Table" to track weak references.
* The memory is fully freed only when weak references are cleared. (Note: This implementation detail changes slightly between Swift versions, but awareness of "Zombie objects" or side tables shows deep research).



---

### **Section 5: Coding Challenge (Whiteboard/IDE)**

**Q9: The "Delegate" Trap.**

* **Prompt:** "Write a simple Delegate pattern in Swift for a `Downloader` class and a `ViewController`."
* **What I'm watching for:**
1. Do you define the protocol as `AnyObject` or `class`?
2. Do you declare the delegate property as `weak`?


```swift
protocol DownloaderDelegate: AnyObject { // Must restrict to class types
    func didDownload()
}

class Downloader {
    weak var delegate: DownloaderDelegate? // MUST be weak
}

```


3. If you fail to make it weak, I will ask: "What happens if the ViewController owns the Downloader, and the Downloader owns the ViewController as a delegate?" (Answer: Strong Reference Cycle -> Memory Leak).



---

### **Summary of Keywords checking for:**

* Strong vs. Weak vs. Unowned
* Reference Cycles (Retain Cycles)
* Capture Lists
* Value Types vs. Reference Types
* Memory Leaks
* `deinit` (how to verify leaks)