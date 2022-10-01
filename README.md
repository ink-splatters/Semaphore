# Semaphore

`Semaphore` is an object that controls access to a resource across multiple execution contexts through use of a traditional counting semaphore.

Unlike [`DispatchSemaphore`](https://developer.apple.com/documentation/dispatch/dispatchsemaphore), `Semaphore` does not block any thread. Instead, it blocks Swift concurrency tasks.

### Usage

```swift
let semaphore = Semaphore(value: 0)

// Block a task until the semaphore is greater than zero
Task {
    await semaphore.wait()
    await doSomething()
}

// Wakes a blocked task
semaphore.signal()
```

Semaphores also provide a way to restrict the access to a limited resource. The sample code below makes sure that `downloadAndSave()` waits until the previous call has completed:

```swift
let semaphore = Semaphore(value: 1)

func downloadAndSave() async throws {
    await semaphore.wait()
    let value = try await downloadValue()
    try await save(value)
    semaphore.signal()
}
```

When you frequently wrap some asynchronous code between `wait()` and `signal()`, you may enjoy the `run` convenience method, which is equivalent:

```swift
func downloadAndSave() async throws {
    await semaphore.run {
        let value = try await downloadValue()
        try await save(value)
    }
}
```