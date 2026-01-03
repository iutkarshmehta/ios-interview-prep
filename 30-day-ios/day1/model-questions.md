### **Topic 1: GCD & async/await**

1. **Conceptual:**
   Explain the difference between **concurrent queues** and **serial queues** in GCD. When would you prefer one over the other?

2. **Practical:**
   You have a function that downloads images for 10 URLs. How would you use **async/await** to fetch all images concurrently and handle errors?

3. **Tricky / edge case:**
   If a **parent task is cancelled** in Swift concurrency, what happens to its **child tasks**? Can you give a scenario where a child might still run even after the parent is cancelled?

---

### **Topic 2: Closures**

1. **Conceptual:**
   Explain **escaping vs non-escaping closures**. Give an example where an escaping closure is required.

2. **Practical:**
   Write a small Swift snippet that uses a **closure to filter an array of integers**, keeping only even numbers.

3. **Tricky / edge case:**
   You have a closure capturing `self` inside a view controller. How do you **avoid retain cycles**? Why does `[weak self]` work here?

---

### **Topic 3: ARC & Reference Counting**

1. **Conceptual:**
   Explain **strong, weak, and unowned references**. When would you use `unowned` instead of `weak`?

2. **Practical:**
   Consider the following Swift code snippet:

```swift
class Node {
    var next: Node?
    init() { print("Node created") }
    deinit { print("Node deallocated") }
}

var node1: Node? = Node()
var node2: Node? = Node()
node1?.next = node2
node2?.next = node1
node1 = nil
node2 = nil
```

What happens here? Why aren’t the nodes deallocated?

3. **Tricky / edge case:**
   Explain a scenario where **weak references may still cause a crash**. How do you prevent it?

---

### **Surprise Topic (I’ll pick one related to your Swift journey)**

**Protocol & Generics**

1. **Conceptual:**
   Explain the difference between `some View` and `any View` in SwiftUI. How does this relate to **opaque vs existential types**?

2. **Practical:**
   Write a generic function that **swaps two elements** in an array, regardless of their type.

3. **Tricky / edge case:**
   Can you have a **protocol with an associated type** used as a variable type? Why or why not?

---


