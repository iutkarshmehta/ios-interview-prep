
## 1. Difference Between `async let` and `TaskGroup`

### Question

What is the difference between:

* `async let`
* `withTaskGroup`

---

## `async let`

`async let` is used to run **a fixed number of asynchronous tasks concurrently**.

### Example

```swift
func load() async {
    async let user = fetchUser()
    async let posts = fetchPosts()

    let result = await (user, posts)
}
```

### Characteristics

* Creates **child tasks**
* Runs tasks **concurrently**
* Number of tasks must be **known at compile time**
* Simple syntax
* Automatically awaited **before leaving scope**

### Use Case

When you have **a few independent async operations**.

Example:

* Fetch user profile
* Fetch posts
* Fetch notifications

---

## `TaskGroup`

`TaskGroup` is used when the **number of tasks is dynamic** or **not known at compile time**.

### Example

```swift
func loadUsers(ids: [Int]) async {

    await withTaskGroup(of: User.self) { group in
        
        for id in ids {
            group.addTask {
                await fetchUser(id)
            }
        }

        for await user in group {
            print(user)
        }
    }
}
```

### Characteristics

* Allows **dynamic task creation**
* Tasks run **concurrently**
* Supports **loops and conditional task creation**
* Parent waits until **all tasks finish**
* Part of **structured concurrency**

### Use Case

When tasks depend on:

* array size
* API response
* user input
* dynamic workloads

---

## Key Differences

| Feature         | `async let`          | `TaskGroup`              |
| --------------- | -------------------- | ------------------------ |
| Number of tasks | Fixed                | Dynamic                  |
| Syntax          | Simple               | More flexible            |
| Task creation   | Static               | Runtime                  |
| Result handling | Direct `await`       | `for await` loop         |
| Use case        | Few concurrent calls | Variable number of tasks |

---

## Quick Interview Summary

> "`async let` is used when you have a fixed number of concurrent async tasks known at compile time, whereas `TaskGroup` allows creating a dynamic number of concurrent tasks and managing them as a group."

---
