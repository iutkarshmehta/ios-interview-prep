Good catch 👍 Let’s keep everything in **clean markdown so you can revise before interviews**.

---

# Swift Concurrency Interview Notes

## Question 2: What happens if you don't `await` an `async let`?

### Question

What happens in this code?

```swift
func load() async {

    async let user = fetchUser()
    async let posts = fetchPosts()

    print("Loading finished")
}
```

Notice:

* `user` is never awaited
* `posts` is never awaited

---

## Correct Answer

Swift **implicitly awaits the child tasks before the scope exits**.

Because `async let` follows **structured concurrency**, the parent function **cannot finish until all child tasks complete**.

---

## Conceptual Execution

```
load()
   ├── async let user
   ├── async let posts
   ↓
print("Loading finished")
   ↓
Swift implicitly awaits user & posts
   ↓
function exits
```

Even though we did not explicitly write:

```swift
await user
await posts
```

Swift ensures they complete before `load()` returns.

---

## Important Rule

> Child tasks created with `async let` **cannot outlive the scope that created them**.

Swift automatically waits for them before exiting the function.

---

## Example with Early Return

```swift
func load() async {

    async let user = fetchUser()
    async let posts = fetchPosts()

    return
}
```

Even here:

```
return
   ↓
Swift implicitly awaits user & posts
   ↓
function exits
```

The function **will not return until both tasks finish**.

---

## Interview-Ready Answer

> "`async let` creates child tasks tied to the scope. If the results are not explicitly awaited, Swift automatically waits for them before exiting the scope to enforce structured concurrency."

---


