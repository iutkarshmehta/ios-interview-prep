

---

# **Swift: Opaque Types and Boxed/Existential Types**

---

## **1️⃣ Opaque Types (`some`)**

### **Definition**

* An **opaque type** is a type whose **exact concrete type is hidden**, but the **interface (protocol conformance) is known**.
* In Swift, we use the keyword `some` to declare **opaque types**.

> Think of it as: “I promise to return something that conforms to this protocol, but I won’t tell you the exact type.”

---

### **Example**

```swift id="opaque1"
protocol Shape {
    func area() -> Double
}

struct Circle: Shape {
    var radius: Double
    func area() -> Double { return .pi * radius * radius }
}

struct Square: Shape {
    var side: Double
    func area() -> Double { return side * side }
}

// Opaque type: hides exact type, exposes only Shape
func makeShape() -> some Shape {
    return Circle(radius: 5)
}
```

* **Key points:**

  * `makeShape()` returns **some Shape**, but caller **cannot know it’s Circle**.
  * Compiler **knows the type**, so Swift can optimize code, but caller **cannot access Circle-specific methods**.

---

### **Use Cases**

* Encapsulation: hide implementation details.
* Return protocol types **without losing type safety**.
* Used a lot in **SwiftUI**, e.g., `some View`.

```swift id="opaque2"
func body() -> some View {
    Text("Hello") // actual type is Text, but hidden
}
```

---

### **Interview Questions on Opaque Types**

1. What is the difference between `some Protocol` and `Protocol` as a return type?
2. Why do we need opaque types instead of just returning a protocol type?
3. Give an example in SwiftUI using `some View`.

**Answer tip:**

* `some Protocol` = opaque type, exact type is **hidden**, **type-safe**, compiler knows.
* `Protocol` (existential) = boxed type, exact type **unknown**, compiler treats as a **generic container**, less optimized.

---

## **2️⃣ Boxed Type / Existential Type (`any`)**

### **Definition**

* A **boxed type** (existential type in Swift 5.6+) is a **protocol type container**.
* Use the `any` keyword to represent **any value conforming to a protocol**, regardless of the concrete type.

> Think of it as a **box**: you know it conforms to a protocol, but the **actual type inside the box can vary at runtime**.

---

### **Example**

```swift id="existential1"
protocol Shape {
    func area() -> Double
}

let shapes: [any Shape] = [
    Circle(radius: 3),
    Square(side: 4)
]

for shape in shapes {
    print(shape.area()) // works for both Circle and Square
}
```

* **Key points:**

  * The **exact type is unknown at compile-time**.
  * Slower than opaque types because Swift uses **dynamic dispatch**.
  * Use `any` keyword in Swift 5.6+: `let shape: any Shape = Circle(radius: 5)`

---

### **Difference Between Opaque and Boxed Types**

| Feature         | Opaque Type (`some`)         | Boxed Type (`any`)                        |
| --------------- | ---------------------------- | ----------------------------------------- |
| Type knowledge  | Compiler knows concrete type | Compiler does NOT know concrete type      |
| Performance     | Optimized, static dispatch   | Dynamic dispatch, slightly slower         |
| Flexibility     | Fixed hidden type            | Can store multiple types in one container |
| Syntax          | `some Protocol`              | `any Protocol`                            |
| SwiftUI example | `some View`                  | `[any View]`                              |

---

### **Interview Questions You May Get**

1. Difference between `some Protocol` and `any Protocol`?

   > `some` = opaque, compiler knows type, type-safe, better performance
   > `any` = existential, runtime type unknown, dynamic dispatch

2. When to use opaque types vs boxed types?

   * **Opaque (`some`)** → when returning a single type hidden for abstraction (SwiftUI, libraries).
   * **Boxed (`any`)** → when you need a **collection of multiple types** conforming to the same protocol.

3. Why does SwiftUI use `some View` instead of `any View`?

   * Compiler knows the exact view type at compile-time → optimized rendering.
   * Using `any View` would require **dynamic dispatch** and reduce performance.

---

### **Quick Mental Model**

* **Opaque (`some`)** = **“I’ll hide the exact type but it’s always the same behind the scenes”**
* **Boxed (`any`)** = **“I don’t know the type at compile-time; treat it as a box conforming to the protocol”**

---

### **Example Combining Both**

```swift id="combine1"
func makeShapes() -> [any Shape] {
    return [
        Circle(radius: 2),
        Square(side: 3)
    ]
}

func makeCircle() -> some Shape {
    return Circle(radius: 5) // opaque, always Circle
}
```

---

### **Interview Summary Answer**

> “In Swift, `some Protocol` is an **opaque type**, meaning the exact type is hidden from the caller, but the compiler knows it and can optimize. This is used when returning a single type but hiding implementation details.
> `any Protocol` is a **boxed type** (existential), meaning the type inside is unknown at compile-time and Swift uses dynamic dispatch. This is useful when you want to store multiple different types conforming to a protocol in one collection.
> Key difference: `some` = type-safe, optimized; `any` = flexible, runtime-checked.”

---


