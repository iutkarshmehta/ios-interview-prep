
**Question 1: Can you explain the difference between strong, weak, and unowned references in Swift? When would you use each one, and what problems can arise if you use the wrong type?**


### **Strong References**
- This is the **default** reference type in Swift
- Creates a strong ownership relationship - the reference keeps the object alive in memory
- ARC (Automatic Reference Counting) increases the retain count by 1

**When to use:** Most of the time! Use strong references when you want to own the object and keep it in memory.

**Problem if misused:** Can cause **retain cycles (memory leaks)** when two objects strongly reference each other.

### **Weak References**
- Does NOT increase the retain count
- Must be declared as **optional** (`var`) because it automatically becomes `nil` when the object is deallocated
- Example: `weak var delegate: MyDelegate?`

**When to use:** 
- Delegate patterns (the delegate doesn't own the delegating object)
- Parent-child relationships where the child references the parent
- Any time you need to reference an object but don't want ownership

**Problem if misused:** If you use strong instead of weak in delegates, you'll get retain cycles. If you use weak when you need strong, the object might be deallocated unexpectedly.

### **Unowned References**
- Also does NOT increase the retain count
- Assumes the object will **always exist** when accessed
- Is **non-optional** - you're guaranteeing the object is alive
- Crashes if you access it after the object is deallocated

**When to use:**
- When you're certain the referenced object has the same or longer lifetime
- Closures that capture `self` when you know `self` won't be deallocated first
- Example: `[unowned self]` in closures

**Problem if misused:** Using unowned when the object might be deallocated will cause a **crash**. Using strong creates retain cycles.

### **Classic Example - Retain Cycle:**

```swift
class Person {
    var apartment: Apartment?
}

class Apartment {
    var tenant: Person?  // Should be weak!
}

let john = Person()
let unit4A = Apartment()
john.apartment = unit4A
unit4A.tenant = john  // Retain cycle! Neither will be deallocated
```

**Fix:** Make `tenant` weak: `weak var tenant: Person?`

**Question 2: In SwiftUI, what's the difference between `@State`, `@Binding`, `@ObservedObject`, and `@StateObject`? When would you use each one?**


### **@State**
- Used for **simple value types** (String, Int, Bool, etc.) that are **private to a view**
- The view **owns** this data
- When the value changes, SwiftUI automatically re-renders the view
- Should be marked as `private`

**Example:**
```swift
struct ContentView: View {
    @State private var isToggled = false
    
    var body: some View {
        Toggle("Switch", isOn: $isToggled)
    }
}
```

---

### **@Binding**
- Creates a **two-way connection** to data owned by another view
- Doesn't own the data - just references it
- Use `$` to pass a binding to a child view
- Think of it as a "reference" to someone else's @State

**Example:**
```swift
struct ParentView: View {
    @State private var text = ""
    
    var body: some View {
        ChildView(text: $text) // Pass binding with $
    }
}

struct ChildView: View {
    @Binding var text: String // Receive binding
    
    var body: some View {
        TextField("Enter text", text: $text)
    }
}
```

---

### **@ObservedObject**
- Used for **reference types** (classes) that conform to `ObservableObject`
- The view does **NOT own** the object - it's created and managed elsewhere
- When `@Published` properties change, the view updates
- The object can be deallocated if the parent stops passing it

**Example:**
```swift
class ViewModel: ObservableObject {
    @Published var count = 0
}

struct ParentView: View {
    @StateObject private var viewModel = ViewModel()
    
    var body: some View {
        ChildView(viewModel: viewModel) // Pass to child
    }
}

struct ChildView: View {
    @ObservedObject var viewModel: ViewModel // Child observes but doesn't own
    
    var body: some View {
        Text("Count: \(viewModel.count)")
    }
}
```

---

### **@StateObject**
- Also used for **reference types** conforming to `ObservableObject`
- The view **owns** and manages the lifecycle of the object
- SwiftUI ensures the object survives view updates/recreations
- Use this when the view should **create and own** the object

**Example:**
```swift
struct ContentView: View {
    @StateObject private var viewModel = ViewModel() // View owns it
    
    var body: some View {
        Text("Count: \(viewModel.count)")
    }
}
```

---

### **Key Differences Summary:**

| Property Wrapper | Ownership | Type | Use Case |
|-----------------|-----------|------|----------|
| @State | View owns it | Value types | Simple local state |
| @Binding | References parent's data | Any | Two-way data flow |
| @ObservedObject | External ownership | ObservableObject | Passed from parent |
| @StateObject | View owns it | ObservableObject | View creates the object |

**Common mistake:** Using `@ObservedObject` when you should use `@StateObject`. If the view creates the object, use `@StateObject` to prevent it from being recreated on every view update.

**Question 3: How does SwiftUI's layout system work? Can you explain the three-step process that SwiftUI uses to size and position views? Also, what's the difference between a flexible frame and a fixed frame?**

No problem! Let me explain SwiftUI's layout system:

### **SwiftUI's 3-Step Layout Process**

SwiftUI uses a three-step process to layout views on screen:

### **Step 1: Parent Proposes a Size to Child**
- The parent view offers a size to its child
- This is a "proposal" - not a command
- Example: A parent might say "I have 300x400 points available for you"

### **Step 2: Child Chooses Its Own Size**
- The child view looks at the proposed size and decides what size it actually wants to be
- The child can choose to be smaller, the same, or sometimes larger than proposed
- Different views have different sizing behaviors

### **Step 3: Parent Places the Child**
- Once the child reports back its chosen size, the parent positions the child within its own coordinate space

This happens recursively for every view in the hierarchy.

---

### **How Different Views Respond to Size Proposals**

**Text:**
- Ignores the proposal and sizes itself based on its content
- Example: `Text("Hello")` will be just big enough for the text

**Image:**
- By default, uses its natural/intrinsic size
- With `.resizable()`, it adapts to the proposed size

**Color / Rectangle:**
- Takes up ALL the space offered (greedy)
- Example: `Color.red` will fill the entire proposed area

**Spacer:**
- Expands to fill available space

---

### **Fixed Frame vs Flexible Frame**

### **Fixed Frame**
```swift
Text("Hello")
    .frame(width: 200, height: 100)
```
- Sets an **exact, fixed size**
- The view will be exactly that size, regardless of content
- Content might be clipped if it's larger

### **Flexible Frame**
```swift
Text("Hello")
    .frame(minWidth: 100, maxWidth: 300, minHeight: 50, maxHeight: 200)
```
- Provides **constraints** rather than exact sizes
- The view can grow/shrink within these bounds based on content and available space
- More flexible and adaptive

### **Special Case - Infinite Max**
```swift
Text("Hello")
    .frame(maxWidth: .infinity)
```
- Tells the view it can expand to fill all available horizontal space
- Still respects the parent's proposal
- Commonly used to make views fill their container

---

### **Practical Example**

```swift
VStack {
    Text("Fixed")
        .frame(width: 200, height: 50) // Exactly 200x50
        .background(Color.blue)
    
    Text("Flexible")
        .frame(minWidth: 100, maxWidth: 300) // Between 100-300 wide
        .background(Color.green)
    
    Text("Fill Width")
        .frame(maxWidth: .infinity) // Expands to fill parent
        .background(Color.red)
}
.padding()
```

---

### **Why This Matters**

Understanding this helps you:
- Debug layout issues (why isn't my view the size I expect?)
- Create responsive designs that adapt to different screen sizes
- Use modifiers like `.layoutPriority()` and `.fixedSize()` effectively

**Common pitfall:** Putting a `Color` or `Rectangle` inside a view and wondering why it takes up the whole screen - it's because they're greedy and take all proposed space!

**Question 4: What is the `@Environment` property wrapper in SwiftUI, and how is it different from `@EnvironmentObject`? Can you give examples of when you'd use each one?**

Great! Let me explain both:

### **@Environment**

- Used to read **system-provided values** from SwiftUI's environment
- These are built-in values that SwiftUI automatically manages
- Examples: color scheme, layout direction, accessibility settings, dismiss action, etc.
- You don't need to manually inject these - they're already there

**Common Examples:**

```swift
struct ContentView: View {
    @Environment(\.colorScheme) var colorScheme // Light or dark mode
    @Environment(\.dismiss) var dismiss // To dismiss a sheet/modal
    @Environment(\.horizontalSizeClass) var sizeClass // Compact or regular
    
    var body: some View {
        VStack {
            if colorScheme == .dark {
                Text("Dark mode is on")
            }
            
            Button("Close") {
                dismiss() // Dismiss current view
            }
        }
    }
}
```

**Other useful @Environment values:**
- `\.locale` - Current locale
- `\.dynamicTypeSize` - Text size accessibility setting
- `\.isEnabled` - Whether view is enabled
- `\.scenePhase` - App lifecycle state

---

### **@EnvironmentObject**

- Used for **custom objects** you want to share across multiple views
- You must **manually inject** it using `.environmentObject()` modifier
- The object must conform to `ObservableObject`
- Useful for sharing data deep in the view hierarchy without passing it through every view
- Think of it as "dependency injection" for SwiftUI

**Example:**

```swift
// 1. Create your model
class UserSettings: ObservableObject {
    @Published var username = ""
    @Published var isDarkMode = false
}

// 2. Inject at the top level
@main
struct MyApp: App {
    @StateObject private var settings = UserSettings()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(settings) // Inject here
        }
    }
}

// 3. Access anywhere in the hierarchy
struct ContentView: View {
    var body: some View {
        NavigationStack {
            ChildView()
        }
    }
}

struct ChildView: View {
    var body: some View {
        GrandchildView()
    }
}

struct GrandchildView: View {
    @EnvironmentObject var settings: UserSettings // Access directly!
    
    var body: some View {
        VStack {
            Text("Hello, \(settings.username)")
            Toggle("Dark Mode", isOn: $settings.isDarkMode)
        }
    }
}
```

---

### **Key Differences**

| Feature | @Environment | @EnvironmentObject |
|---------|-------------|-------------------|
| **Purpose** | System values | Custom objects |
| **Injection** | Automatic by SwiftUI | Manual with `.environmentObject()` |
| **Type** | Value types mostly | ObservableObject classes |
| **Examples** | colorScheme, dismiss | UserSettings, AppState |
| **Crash if missing?** | No, has defaults | Yes! Must be injected |

---

### **When to Use Each**

**Use @Environment when:**
- You need system-provided values (color scheme, dismiss, etc.)
- You want to read built-in SwiftUI environment settings

**Use @EnvironmentObject when:**
- You have app-wide state (user authentication, settings, themes)
- You want to avoid prop drilling (passing data through many layers)
- Multiple unrelated views need access to the same data
- You're building a shared dependency like a network service or database

---

### **Important Warning**

If you use `@EnvironmentObject` but forget to inject it, your app will **crash** with a message like:
```
Fatal error: No ObservableObject of type UserSettings found
```

To avoid this, always inject it at a parent level before any child tries to access it.

---

### **Bonus: Custom Environment Values**

You can also create your own custom `@Environment` keys (not objects):

```swift
// Define custom key
private struct ThemeKey: EnvironmentKey {
    static let defaultValue: String = "default"
}

extension EnvironmentValues {
    var theme: String {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

// Use it
struct ParentView: View {
    var body: some View {
        ChildView()
            .environment(\.theme, "ocean")
    }
}

struct ChildView: View {
    @Environment(\.theme) var theme
    
    var body: some View {
        Text("Theme: \(theme)")
    }
}
```

**Question 5: What are some techniques to optimize performance in SwiftUI? For example, how would you handle a List with thousands of items, or prevent unnecessary view updates?**

Exactly! `LazyVStack` is one technique, but let me give you a comprehensive overview of SwiftUI performance optimization:

### **1. Lazy Stacks and Lists**

### **LazyVStack / LazyHStack**
```swift
ScrollView {
    LazyVStack {
        ForEach(0..<10000, id: \.self) { index in
            RowView(index: index)
        }
    }
}
```
- Creates views **only when they're about to appear** on screen
- Regular `VStack` creates ALL views immediately (bad for large datasets)
- Dramatically reduces memory usage and initial load time

### **List**
```swift
List(items) { item in
    ItemRow(item: item)
}
```
- Built-in lazy loading and recycling (like UITableView)
- Best choice for large datasets
- Automatically handles scrolling performance

---

### **2. Preventing Unnecessary View Updates**

### **Equatable Views**
```swift
struct ItemRow: View, Equatable {
    let item: Item
    
    var body: some View {
        Text(item.name)
    }
    
    // Define what makes two views equal
    static func == (lhs: ItemRow, rhs: ItemRow) -> Bool {
        lhs.item.id == rhs.item.id
    }
}

// Use with .equatable()
List(items) { item in
    ItemRow(item: item)
        .equatable() // Only re-renders if ItemRow changes
}
```
- SwiftUI will skip re-rendering if the view hasn't actually changed
- Useful when parent updates but child data stays the same

---

### **3. Splitting Large ViewModels**

**Bad - One massive ViewModel:**
```swift
class AppViewModel: ObservableObject {
    @Published var user: User
    @Published var products: [Product]
    @Published var cart: Cart
    @Published var settings: Settings
}

struct ProductListView: View {
    @ObservedObject var viewModel: AppViewModel
    // Updates even when only user changes!
}
```

**Good - Separate ViewModels:**
```swift
class ProductViewModel: ObservableObject {
    @Published var products: [Product]
}

class UserViewModel: ObservableObject {
    @Published var user: User
}

struct ProductListView: View {
    @ObservedObject var viewModel: ProductViewModel
    // Only updates when products change
}
```

---

### **4. Use @Published Carefully**

**Bad:**
```swift
class ViewModel: ObservableObject {
    @Published var largeArray: [Item] = [] // Every change triggers update
    
    func addItem(_ item: Item) {
        largeArray.append(item) // Entire view updates!
    }
}
```

**Better:**
```swift
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    var cachedData: [String: Any] = [:] // Not @Published - won't trigger updates
    
    func updateCache() {
        // Do expensive work without triggering view updates
        cachedData = processData()
    }
}
```

**Only mark properties as `@Published` if views need to react to their changes.**

---

### **5. Avoid Heavy Computations in Body**

**Bad:**
```swift
struct ContentView: View {
    let items: [Item]
    
    var body: some View {
        List(items) { item in
            // This runs EVERY time body is called!
            let processed = heavyProcessing(item)
            Text(processed)
        }
    }
}
```

**Good:**
```swift
struct ContentView: View {
    let items: [Item]
    
    // Computed property cached between body calls
    private var processedItems: [String] {
        items.map { heavyProcessing($0) }
    }
    
    var body: some View {
        List(Array(zip(items, processedItems)), id: \.0.id) { item, processed in
            Text(processed)
        }
    }
}
```

Or better yet, do processing in the ViewModel before passing to the view.

---

### **6. Use @State vs @StateObject Correctly**

**Bad:**
```swift
struct ParentView: View {
    var body: some View {
        ChildView()
    }
}

struct ChildView: View {
    @StateObject var viewModel = ViewModel() // WRONG! Recreated every time
    
    var body: some View {
        Text("Count: \(viewModel.count)")
    }
}
```

If `ParentView` re-renders, `@StateObject` in `ChildView` is recreated!

**Good:**
```swift
struct ParentView: View {
    @StateObject private var viewModel = ViewModel()
    
    var body: some View {
        ChildView(viewModel: viewModel)
    }
}

struct ChildView: View {
    @ObservedObject var viewModel: ViewModel // Receives from parent
    
    var body: some View {
        Text("Count: \(viewModel.count)")
    }
}
```

---

### **7. Task and AsyncImage Optimization**

**Bad:**
```swift
struct ImageRow: View {
    let url: URL
    @State private var image: UIImage?
    
    var body: some View {
        Image(uiImage: image ?? UIImage())
            .task {
                // Fires every time view appears
                image = await loadImage(url)
            }
    }
}
```

**Good:**
```swift
struct ImageRow: View {
    let url: URL
    
    var body: some View {
        AsyncImage(url: url) { image in
            image.resizable()
        } placeholder: {
            ProgressView()
        }
        // Built-in caching and cancellation
    }
}
```

Or use a proper image caching library like Kingfisher or SDWebImage.

---

### **8. Identify Performance Issues**

Use Instruments to profile:
- **Time Profiler**: Find slow body computations
- **Allocations**: Check for memory leaks
- **SwiftUI View Body**: See which views update most frequently

---

### **Quick Checklist for Performance:**

✅ Use `List` or `LazyVStack` for large datasets  
✅ Split large ViewModels into smaller ones  
✅ Only mark necessary properties as `@Published`  
✅ Avoid heavy work in `body`  
✅ Use `.equatable()` for complex child views  
✅ Cache expensive computations  
✅ Use proper state management (@StateObject in owner, @ObservedObject in children)  
✅ Profile with Instruments to find real bottlenecks  

The golden rule: **Don't optimize prematurely!** Profile first, then optimize what's actually slow.

**Question 6: What's the difference between `async/await` and completion handlers in Swift? Also, can you explain what actors are and how they help with concurrency?**

Sure! Let me break down modern Swift concurrency:

### **Completion Handlers vs Async/Await**

### **Old Way - Completion Handlers**

```swift
func fetchUser(id: String, completion: @escaping (Result<User, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            completion(.failure(error))
            return
        }
        
        guard let data = data else {
            completion(.failure(NetworkError.noData))
            return
        }
        
        do {
            let user = try JSONDecoder().decode(User.self, from: data)
            completion(.success(user))
        } catch {
            completion(.failure(error))
        }
    }.resume()
}

// Usage - nested callbacks (pyramid of doom)
fetchUser(id: "123") { result in
    switch result {
    case .success(let user):
        fetchPosts(for: user) { result in
            switch result {
            case .success(let posts):
                fetchComments(for: posts) { result in
                    // More nesting...
                }
            case .failure(let error):
                print(error)
            }
        }
    case .failure(let error):
        print(error)
    }
}
```

**Problems:**
- Callback hell / pyramid of doom
- Easy to forget to call completion
- Can accidentally call completion multiple times
- Hard to handle errors consistently
- Difficult to reason about execution flow

---

### **New Way - Async/Await**

```swift
func fetchUser(id: String) async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    let user = try JSONDecoder().decode(User.self, from: data)
    return user
}

// Usage - clean, linear code
do {
    let user = try await fetchUser(id: "123")
    let posts = try await fetchPosts(for: user)
    let comments = try await fetchComments(for: posts)
    // Use comments
} catch {
    print("Error: \(error)")
}
```

**Benefits:**
- Linear, easy-to-read code
- Automatic error propagation with `throws`
- Compiler enforces correct usage
- Can use regular control flow (if, for, while)
- Automatic cancellation support

---

### **Key Async/Await Concepts**

### **async Function**
```swift
func loadData() async -> Data {
    // Can call other async functions with await
    let result = await networkCall()
    return result
}
```

### **await Keyword**
- Marks a **suspension point** where the function can pause
- Thread is freed up to do other work while waiting
- When result is ready, function resumes

### **Task**
```swift
// In SwiftUI
.task {
    // Runs async code
    data = try await fetchData()
}

// Create detached task
Task.detached {
    await doBackgroundWork()
}

// Create task with priority
Task(priority: .high) {
    await urgentWork()
}
```

### **Async Let (Parallel Execution)**
```swift
// Sequential (slow)
let user = try await fetchUser()
let posts = try await fetchPosts()
let settings = try await fetchSettings()

// Parallel (fast) - all run simultaneously
async let user = fetchUser()
async let posts = fetchPosts()
async let settings = fetchSettings()

let results = try await (user, posts, settings)
```

---

### **Actors - Thread-Safe by Design**

Actors are reference types that protect their state from data races.

### **The Problem - Data Races**

```swift
class Counter {
    var count = 0
    
    func increment() {
        count += 1 // NOT thread-safe!
    }
}

let counter = Counter()

// Multiple threads accessing at once - CRASH or wrong values
Task { counter.increment() }
Task { counter.increment() }
Task { counter.increment() }
```

---

### **The Solution - Actors**

```swift
actor Counter {
    var count = 0
    
    func increment() {
        count += 1 // Thread-safe automatically!
    }
    
    func getCount() -> Int {
        return count
    }
}

let counter = Counter()

// All accesses are serialized automatically
Task {
    await counter.increment() // Must use await
}

Task {
    await counter.increment()
}

Task {
    let value = await counter.getCount()
    print(value)
}
```

**How Actors Work:**
- Only one task can access the actor's mutable state at a time
- All methods/properties must be accessed with `await` (except from within the actor itself)
- Swift compiler enforces this at compile time
- No more data races!

---

### **Actor Isolation**

```swift
actor BankAccount {
    private var balance: Double = 0
    
    func deposit(_ amount: Double) {
        balance += amount // Can access directly inside actor
    }
    
    func withdraw(_ amount: Double) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount
        return true
    }
    
    // Synchronous access within actor
    func transfer(to other: BankAccount, amount: Double) async {
        if withdraw(amount) { // No await needed - same actor
            await other.deposit(amount) // Await needed - different actor
        }
    }
}
```

---

### **MainActor - UI Thread Safety**

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var data: [Item] = []
    
    // This method always runs on main thread
    func updateUI() {
        data.append(Item())
    }
}

// Or mark individual functions
class DataManager {
    @MainActor
    func updateUI() {
        // Guaranteed to run on main thread
    }
    
    func fetchData() async {
        let result = await networkCall()
        
        // Must explicitly jump to main actor for UI updates
        await MainActor.run {
            // Update UI here
        }
    }
}
```

---

### **Actor vs Class**

| Feature | Class | Actor |
|---------|-------|-------|
| **Thread Safety** | Manual (locks, queues) | Automatic |
| **Access** | Direct | Must use `await` |
| **Inheritance** | Yes | No |
| **Data Races** | Possible | Prevented by compiler |
| **Performance** | Faster (no serialization) | Slightly slower |

---

### **Common Patterns**

### **Converting Completion Handler to Async**

```swift
// Old API
func oldFetch(completion: @escaping (Result<Data, Error>) -> Void) {
    // ...
}

// Wrap it
func fetch() async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        oldFetch { result in
            continuation.resume(with: result)
        }
    }
}
```

### **Task Cancellation**

```swift
let task = Task {
    for i in 1...100 {
        try Task.checkCancellation() // Throws if cancelled
        await doWork(i)
    }
}

// Cancel from somewhere else
task.cancel()
```

---

## **Real-World Example**

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]
    
    func image(for url: URL) -> UIImage? {
        cache[url]
    }
    
    func store(_ image: UIImage, for url: URL) {
        cache[url] = image
    }
}

@MainActor
class ImageViewModel: ObservableObject {
    @Published var image: UIImage?
    private let cache = ImageCache()
    
    func loadImage(from url: URL) async {
        // Check cache (actor-isolated)
        if let cached = await cache.image(for: url) {
            image = cached
            return
        }
        
        // Download
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            guard let downloaded = UIImage(data: data) else { return }
            
            // Store in cache
            await cache.store(downloaded, for: url)
            
            // Update UI (already on MainActor)
            image = downloaded
        } catch {
            print("Failed to load image: \(error)")
        }
    }
}
```

---

### **Key Takeaways**

✅ Async/await makes asynchronous code look synchronous  
✅ Actors prevent data races automatically  
✅ Use `@MainActor` for UI updates  
✅ `async let` for parallel execution  
✅ Tasks for creating concurrent work  
✅ Compiler enforces thread safety  


**Question 7: What is the Combine framework? Can you explain the difference between a Publisher, Subscriber, and Operator? Also, how does Combine compare to async/await?**

### What is Combine?

Combine is Apple's **reactive programming framework** for handling asynchronous events and data streams over time. Think of it as a way to react to values that change over time.

### Core Concepts

### Publisher
- **Emits values over time**
- Like a radio station broadcasting signals
- Can emit zero, one, or multiple values
- Eventually completes (successfully or with an error)

```swift
// Simple example
let publisher = [1, 2, 3, 4, 5].publisher
```

### Subscriber
- **Receives values from a Publisher**
- Like a radio receiver tuning into a station
- Requests values and processes them

```swift
publisher.sink { value in
    print(value) // Prints 1, 2, 3, 4, 5
}
```

### Operator
- **Transforms values between Publisher and Subscriber**
- Like filters or processors on the data stream
- Examples: map, filter, debounce, etc.

```swift
[1, 2, 3, 4, 5].publisher
    .map { $0 * 2 }        // Multiply each by 2
    .filter { $0 > 5 }     // Only keep values > 5
    .sink { print($0) }    // Prints: 6, 8, 10
```

### Real SwiftUI Example

```swift
class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [String] = []
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        // React to search text changes
        $searchText
            .debounce(for: 0.5, scheduler: RunLoop.main) // Wait 0.5s after typing stops
            .removeDuplicates() // Ignore if same as previous
            .sink { [weak self] text in
                self?.performSearch(text)
            }
            .store(in: &cancellables) // Keep subscription alive
    }
    
    func performSearch(_ query: String) {
        // Perform search
        results = ["Result 1", "Result 2"]
    }
}
```

### Common Publishers

### Just
Emits a single value then completes:
```swift
Just(42)
    .sink { print($0) } // Prints: 42
```

### Future
For single async operations:
```swift
Future<String, Error> { promise in
    // Do async work
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        promise(.success("Done!"))
    }
}
```

### @Published
Automatically creates a publisher from a property:
```swift
class ViewModel: ObservableObject {
    @Published var count = 0 // Creates a publisher
}

let vm = ViewModel()
vm.$count // Access the publisher with $
    .sink { print("Count is now: \($0)") }
```

### Timer
Emits values periodically:
```swift
Timer.publish(every: 1.0, on: .main, in: .common)
    .autoconnect()
    .sink { print("Tick: \($0)") }
```

### Common Operators

### map
Transform each value:
```swift
[1, 2, 3].publisher
    .map { $0 * 10 }
    .sink { print($0) } // 10, 20, 30
```

### filter
Only pass certain values:
```swift
[1, 2, 3, 4, 5].publisher
    .filter { $0 % 2 == 0 }
    .sink { print($0) } // 2, 4
```

### debounce
Wait for a pause in events:
```swift
searchTextField.textPublisher
    .debounce(for: 0.3, scheduler: RunLoop.main)
    .sink { searchText in
        // Only fires 0.3s after user stops typing
    }
```

### flatMap
Switch to a new publisher for each value:
```swift
userIDPublisher
    .flatMap { userID in
        fetchUserDetails(userID) // Returns another publisher
    }
    .sink { userDetails in
        print(userDetails)
    }
```

### combineLatest
Combine values from multiple publishers:
```swift
let firstName = PassthroughSubject<String, Never>()
let lastName = PassthroughSubject<String, Never>()

firstName.combineLatest(lastName)
    .map { "\($0) \($1)" }
    .sink { print($0) }

firstName.send("John")
lastName.send("Doe") // Prints: "John Doe"
```

### Network Request Example

```swift
class NetworkManager {
    func fetchUsers() -> AnyPublisher<[User], Error> {
        URLSession.shared
            .dataTaskPublisher(for: url)
            .map(\.data) // Extract data
            .decode(type: [User].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main) // Switch to main thread
            .eraseToAnyPublisher() // Hide implementation details
    }
}

// Usage
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    private var cancellables = Set<AnyCancellable>()
    
    func loadUsers() {
        NetworkManager().fetchUsers()
            .sink(
                receiveCompletion: { completion in
                    switch completion {
                    case .finished:
                        print("Success")
                    case .failure(let error):
                        print("Error: \(error)")
                    }
                },
                receiveValue: { [weak self] users in
                    self?.users = users
                }
            )
            .store(in: &cancellables)
    }
}
```

### Memory Management

Always store subscriptions or they'll be cancelled immediately:
```swift
private var cancellables = Set<AnyCancellable>()

publisher
    .sink { value in
        print(value)
    }
    .store(in: &cancellables) // Important!
```

### Combine vs Async/Await

| Feature | Combine | Async/Await |
|---------|---------|-------------|
| **Use Case** | Multiple values over time | Single async operations |
| **Complexity** | More complex, learning curve | Simple, readable |
| **Cancellation** | Built-in with AnyCancellable | Manual with Task |
| **Operators** | Rich set (map, filter, debounce) | Limited (use loops/conditionals) |
| **SwiftUI** | @Published works naturally | Need .task {} modifier |
| **Best For** | Reactive streams, UI bindings | Network calls, single async tasks |

### When to Use Each

**Use Combine when:**
- Reacting to UI changes (text fields, toggles)
- Multiple values over time (timers, notifications)
- Complex data transformations with operators
- Need debouncing, throttling, combining streams

**Use Async/Await when:**
- Single network requests
- One-time async operations
- Simple sequential async code
- Modern iOS 15+ projects

### Quick Example Comparison

**Combine:**
```swift
class ViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [Item] = []
    
    init() {
        $searchText
            .debounce(for: 0.5, scheduler: RunLoop.main)
            .flatMap { text in
                self.search(text)
            }
            .assign(to: &$results)
    }
}
```

**Async/Await:**
```swift
class ViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [Item] = []
    
    func searchDebounced() async {
        try? await Task.sleep(nanoseconds: 500_000_000)
        results = await search(searchText)
    }
}
```

### Key Takeaways

- Combine is for **reactive programming** with data streams
- Publisher emits, Subscriber receives, Operators transform
- Great for UI reactive patterns
- Modern apps often use **both** Combine and async/await together
- Async/await is simpler for one-time operations

