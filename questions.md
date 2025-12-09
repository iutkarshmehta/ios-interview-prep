# 100 iOS Interview Questions (3 Years Exp) - Dec 2025

**Target Profile:** Mid-Level / Early Senior
**Focus:** Swift 6, Modern Concurrency, Hybrid UIKit/SwiftUI, and Component Design.

---

## Part 1: Swift Language & Internals (Questions 1–20)
*Core mechanics. Passing the initial screen requires deep knowledge here.*

1.  **[Hot]** What is the difference between `Sendable` and `@unchecked Sendable`? When would you use each?
2.  Explain **Automatic Reference Counting (ARC)**. How is it different from Garbage Collection?
3.  What is a **Retain Cycle**? How do you debug it using Xcode Instruments?
4.  What is the difference between `weak` and `unowned` references? When is `unowned` safe to use?
5.  Explain **Copy-on-Write (CoW)**. Which Swift types use it, and how can you implement it for your own custom struct?
6.  What are **Generics**? How does `some View` (Opaque Types) differ from `any View` (Existential Types)?
7.  What is **Method Dispatch**? Explain Static vs. Dynamic vs. Message Dispatch.
8.  What is the difference between `static` and `class` keywords for properties/methods?
9.  How does `defer` work? What is the order of execution if you have multiple defer blocks?
10. What is the purpose of `Result` type in Swift?
11. Explain **Codable**. How do you handle a JSON key that doesn't match your property name?
12. What are **Property Wrappers**? How would you implement a custom `@UserDefault` wrapper?
13. What is the difference between `Struct` and `Class`? (Focus on: stack vs. heap, pass-by-value vs. reference).
14. What are **Associated Types** in protocols?
15. What is the **Main Actor** (`@MainActor`)? Why do we need it for UI updates?
16. How do **Extensions** work? Can you add stored properties to an extension? (Why/Why not?)
17. What is **Key-Value Observing (KVO)**? Is it still relevant in Swift?
18. What is the difference between `Sequence` and `Collection` protocols?
19. Explain **Error Handling** in Swift. `try?` vs `try!` vs `do-catch`.
20. What is **Access Control** (`open`, `public`, `internal`, `fileprivate`, `private`)? When do you use `open`?

---

## Part 2: Modern Concurrency & Swift 6 (Questions 21–35)
*This is the differentiator section for late 2025 interviews.*

21. **[Hot]** How do **Actors** prevent data races? How are they different from classes with locks?
22. **[Hot]** What is the new **Strict Concurrency Checking** in Swift 6?
23. What is the difference between `Task`, `Task.detached`, and `TaskGroup`?
24. How do you migrate a completion-handler based API to `async/await` using `CheckedContinuation`?
25. What is **Structured Concurrency**? What happens to child tasks if a parent task is cancelled?
26. Explain `async let` vs. `await`. When do they run in parallel?
27. What is the difference between GCD (Grand Central Dispatch) and Swift Concurrency?
28. How do you handle thread safety without using Actors? (e.g., `NSLock`, `DispatchQueue` barriers).
29. What is the `nonisolated` keyword used for in an Actor?
30. How do you debug a **Deadlock**?
31. What is the Global Concurrent Executor?
32. How does `AsyncSequence` work? Give a practical example (e.g., notification stream).
33. **[Hot]** What does the compiler error "Capture of non-sendable type... in a `@Sendable` closure" mean?
34. How do you optimize high-performance code using `withUnsafeContinuation`?
35. What is `DispatchQueue.main.async` equivalent in modern concurrency?

---

## Part 3: UIKit vs. SwiftUI (Questions 36–60)
*Mid-level engineers often manage hybrid apps and need to know both.*

36. **[Hot]** Explain the **Observation Framework** (iOS 17+) vs. `ObservableObject`/`@Published`.
37. What is the **Life Cycle** of a SwiftUI View? (`onAppear`, `task`, `onChange`).
38. How does SwiftUI's diffing algorithm work? (Why are `Identifiable` and stable `id`s important?)
39. Explain `UIViewRepresentable`. How do you pass data *from* UIKit *to* SwiftUI and vice-versa?
40. What is `NavigationStack`? How is it different from `NavigationView`?
41. What is the difference between `@State`, `@Binding`, `@StateObject`, and `@ObservedObject`?
42. **Scenario:** You have a SwiftUI List that lags when scrolling. How do you debug and fix it?
43. How does Auto Layout work? Explain **Intrinsic Content Size** and **Compression Resistance**.
44. What is the **Responder Chain** in UIKit? How do touches bubble up?
45. Explain the View Controller Lifecycle (`viewDidLoad`, `viewWillAppear`, `viewDidLayoutSubviews`, etc.).
46. What is `UICollectionViewDiffableDataSource`? Why use it over the old `dataSource` methods?
47. How do you handle **Keyboard Management** in SwiftUI vs. UIKit?
48. What is the difference between `frame` and `bounds`?
49. How do you create a custom generic view modifier in SwiftUI?
50. What is **contentUnavailableConfiguration** in UIKit (iOS 17+)?
51. How do you implement **Dark Mode** support efficiently?
52. **[Hot]** How do you structure navigation in a large SwiftUI app? (Router pattern / FlowStacks).
53. What is `@Environment`? How do you inject custom values into it?
54. Explain `GeometryReader`. Why is it often considered "dangerous" or a bad practice if overused?
55. How do `CALayer` and `UIView` relate? Which handles touch events?
56. What is **Off-screen Rendering**? Why does it affect performance?
57. How do you implement Accessibility (VoiceOver) in SwiftUI?
58. What is `PreferenceKey` in SwiftUI?
59. How do you force a SwiftUI view to redraw?
60. **Scenario:** A designer wants a button that glows and bounces. Do you use Core Animation, Lottie, or SwiftUI animation? Why?

---

## Part 4: Architecture & Design Patterns (Questions 61–75)
*Moving from "writing code" to "designing code".*

61. Compare **MVVM** vs. **MVC** vs. **VIPER**. Which do you prefer and why?
62. What is the **Coordinator Pattern**? How does it solve the "massive view controller" problem?
63. What is **Dependency Injection**? Explain Constructor Injection vs. Property Injection.
64. How do you create a **Modular Architecture** using Swift Package Manager (SPM)?
65. What is the **Singleton** pattern? Why is it controversial?
66. Explain the **Factory Pattern**. When would you use it in iOS?
67. What is **Clean Architecture**? What are the layers?
68. How do you handle **Deep Linking** in a modular app?
69. What is **Protocol Oriented Programming (POP)**? How does it differ from OOP?
70. **Scenario:** You need to share code between an iOS App, a Widget, and a Watch App. How do you structure this?
71. What is the **Decorator Pattern**? (e.g., standard library Delegation is often a form of this).
72. How do you prevent **Massive ViewModels** in MVVM?
73. What is **Unidirectional Data Flow** (UDF)? (Like The Composable Architecture - TCA).
74. How do you handle analytics tracking without cluttering your View Controllers?
75. What is the "Facade" pattern and where does iOS SDK use it?

---

## Part 5: Data, Networking & Storage (Questions 76–85)

76. How do you design a **Thread-Safe** network layer?
77. Explain **Core Data** stack (Context, Persistent Store, Coordinator).
78. What is `NSCache`? How is it different from `Dictionary`?
79. How do you handle **Certificate Pinning** for security?
80. What is the difference between `URLSession` default, ephemeral, and background configurations?
81. How do you optimize JSON parsing speed for large datasets?
82. **SQLite** vs. **Core Data** vs. **Realm** vs. **SwiftData**. Pros/Cons?
83. How do you handle **Offline Mode** and data synchronization?
84. What is the `Codable` distinction between `encode(to:)` and `init(from:)`?
85. How do you secure sensitive data? (Keychain vs. UserDefaults).

---

## Part 6: Mobile System Design (Questions 86–95)
*Specific component design questions expected at 3 YOE.*

86. **Design an Image Loading Library.** (Key points: Caching (Memory/Disk), De-duplication of requests, Prefetching, cancelling old requests on scroll).
87. **Design a Logging System.** (Key points: Console vs. Remote, Log levels, Performance impact, formatting).
88. **Design a "Feed" View.** (Key points: Pagination, DiffableDataSource, Prefetching, Cell reuse).
89. **Design an Analytics SDK.** (Key points: Batching events, Battery usage, Thread safety).
90. **Design a Search Typeahead.** (Key points: Debouncing, Throttling, Cancelling stale requests).
91. How would you reduce the **App Launch Time**? (Dyld, static vs dynamic linking, lazy initialization).
92. How do you reduce **App Size** (Thinning)?
93. How do you debug a **Battery Drain** issue reported by a user?
94. How do you handle **Database Migration** in a live app?
95. **Scenario:** The app crashes only on Release builds but works on Debug. What do you do?

---

## Part 7: Testing & CI/CD (Questions 96–100)

96. What is the difference between **Unit Tests** and **UI Tests**?
97. How do you mock a Network Service protocol for testing?
98. What is **Snapshot Testing**? When is it useful?
99. Explain the basics of **Fastlane**. What is it used for?
100. How do you configure a CI/CD pipeline (e.g., GitHub Actions/Bitrise) to run tests on every PR?