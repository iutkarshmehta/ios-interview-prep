Here is the text converted to Markdown, organized by sections. I have also added the **Solutions** section at the bottom to help with your practice.

# Home Work - Control Flow & Functions

## 🔁 PART 1: Control Flow – Loops & Conditions

1. Write a program to print numbers from 1 to 10 using a `for-in` loop.
2. Write a program to calculate the sum of all numbers in an array using a loop.
3. Given an array of integers, print only the even numbers using `if` conditions.
4. Write a program to find the largest number in an array.
5. Use a `while` loop to print numbers from 10 to 1.
6. Write a program that checks whether a number is positive, negative, or zero.
7. Given a string, count the number of vowels using a loop.
8. Write a program to stop iterating when a specific value is found using `break`.
9. Use `continue` to skip odd numbers while iterating from 1 to 20.
10. Write a program to compare two numbers using `if-else`.

## 🔀 PART 2: Switch Statements

1. Write a switch statement to print the day of the week based on a number (1–7).
2. Use `switch` to check whether a character is a vowel or consonant.
3. Write a switch statement that handles multiple matching values in one case.
4. Use a switch statement with a `default` case.
5. Write a switch statement using range matching.
6. Use `switch` to categorize a temperature value (cold, warm, hot).
7. Demonstrate how `fallthrough` works in a switch statement.
8. Write a switch statement using tuples.
9. Use `switch` with `where` clauses.
10. Compare `if-else` and `switch` by solving the same problem using both.

## 🔂 PART 3: Control Transfer Statements

1. Write a program that exits a loop early using `break`.
2. Write a program that skips an iteration using `continue`.
3. Use `return` to exit a function early based on a condition.
4. Write a program using `guard` to validate input before continuing execution.

## 🧩 PART 4: Functions – Basics

1. Write a function that takes two integers and returns their sum.
2. Write a function that checks whether a number is even or odd.
3. Create a function that returns the square of a number.
4. Write a function that takes a string and returns its length.
5. Write a function with no parameters and no return value.
6. Write a function that prints a greeting message for a given name.
7. Create a function that returns multiple values using a tuple.
8. Write a function with external and internal parameter names.
9. Write a function that accepts an array and returns the largest element.
10. Write a function that returns a Boolean value.

## 🧠 PART 5: Functions – Advanced Basics

1. Write a function with default parameter values.
2. Write a function that accepts a variable number of parameters (variadic parameters).
3. Write a function that uses `inout` parameters.
4. Write a function that returns an optional value.
5. Write a function that uses `guard` statements for validation.
6. Write a function that calls another function internally.
7. Write a function that demonstrates function overloading.
8. Write a recursive function to calculate the factorial.

## 🔄 PART 6: Real iOS-Oriented Practice

1. Write a function to validate user input (non-empty, valid length).
2. Write a function to calculate total price including tax.
3. Write a function that processes an array and returns filtered results.

---

# Solutions (Swift)

### PART 1: Control Flow

```swift
// 1. 1 to 10 Loop
for i in 1...10 { print(i) }

// 2. Sum of array
let nums = [1, 2, 3, 4]
var sum = 0
for n in nums { sum += n }

// 3. Print evens
for n in nums { if n % 2 == 0 { print(n) } }

// 4. Find Largest
let maxNum = nums.max() ?? 0

// 5. While loop 10 to 1
var count = 10
while count >= 1 {
    print(count)
    count -= 1
}

// 6. Positive/Negative/Zero
let number = -5
if number > 0 { print("Positive") }
else if number < 0 { print("Negative") }
else { print("Zero") }

// 7. Count Vowels
let str = "Apple"
let vowels = "aeiouAEIOU"
var vCount = 0
for char in str { if vowels.contains(char) { vCount += 1 } }

// 8. Break loop
for i in 1...10 {
    if i == 5 { break }
    print(i)
}

// 9. Continue (Skip odds)
for i in 1...20 {
    if i % 2 != 0 { continue }
    print(i)
}

```

### PART 2: Switch Statements

```swift
// 1. Day of week
let day = 3
switch day {
case 1: print("Sunday")
case 2: print("Monday")
default: print("Other Day")
}

// 2 & 3. Vowel/Consonant (Multiple matches)
let char: Character = "a"
switch char {
case "a", "e", "i", "o", "u": print("Vowel")
default: print("Consonant")
}

// 5. Range Matching
let score = 85
switch score {
case 0..<50: print("Fail")
case 50...100: print("Pass")
default: print("Invalid")
}

// 7. Fallthrough
let num = 5
switch num {
case 5:
    print("Is 5")
    fallthrough
default:
    print("This also prints")
}

// 8. Tuples
let point = (1, 1)
switch point {
case (0, 0): print("Origin")
case (_, 0): print("On X-axis")
case (0, _): print("On Y-axis")
default: print("Somewhere else")
}

// 9. Where clause
let val = 10
switch val {
case let x where x % 2 == 0: print("Even \(x)")
default: print("Odd")
}

```

### PART 3: Control Transfer

```swift
// 3. Return early
func check(val: Int) {
    if val < 0 { return }
    print(val)
}

// 4. Guard
func validate(name: String?) {
    guard let n = name, !n.isEmpty else {
        print("Invalid")
        return
    }
    print("Hello \(n)")
}

```

### PART 4: Functions – Basics

```swift
// 1. Sum
func add(_ a: Int, _ b: Int) -> Int { return a + b }

// 7. Multiple values (Tuple)
func getMinMax(arr: [Int]) -> (min: Int, max: Int)? {
    guard let min = arr.min(), let max = arr.max() else { return nil }
    return (min, max)
}

// 8. External/Internal names
func greet(to name: String) { print("Hi \(name)") }
// Usage: greet(to: "John")

```

### PART 5: Functions – Advanced

```swift
// 1. Default Parameter
func log(message: String, level: String = "INFO") {
    print("[\(level)] \(message)")
}

// 2. Variadic Parameters
func sumAll(_ numbers: Int...) -> Int {
    return numbers.reduce(0, +)
}

// 3. Inout Parameter
func swapValues(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}
// Usage: swapValues(&x, &y)

// 8. Recursion (Factorial)
func factorial(_ n: Int) -> Int {
    return n == 0 ? 1 : n * factorial(n - 1)
}

```

### PART 6: Real iOS Practice

```swift
// 1. Validate Input
func validateInput(_ text: String) -> Bool {
    return !text.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty && text.count >= 3
}

// 2. Total Price with Tax
func calculateTotal(price: Double, taxRate: Double) -> Double {
    return price + (price * taxRate)
}

```