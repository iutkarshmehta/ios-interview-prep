## Part 2: Modern Concurrency & Swift 6 (Questions 21–35)

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