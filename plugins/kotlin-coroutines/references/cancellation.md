# Cancellation

## Cancellation Is Cooperative

`cancel()` does not forcibly stop a coroutine. It sends a cancellation signal. The coroutine actually stops only when the code reaches a suspension point or actively checks cancellation state.

Long loops, heavy computation, and polling must support cancellation explicitly:

```kotlin
while (isActive) {
    process()
    yield()
}
```

For batch processing or CPU loops, use:

```kotlin
for (item in items) {
    coroutineContext.ensureActive()
    process(item)
}
```

## cancel and cancelAndJoin

`job.cancel()` only requests cancellation; it does not guarantee that the task has exited. Use `cancelAndJoin()` when the next step depends on cleanup, file closing, listener removal, or state release.

```kotlin
job.cancelAndJoin()
releaseResource()
```

## CancellationException

`CancellationException` is part of the coroutine cancellation protocol, not a normal business failure. `catch (Exception)`, `catch (Throwable)`, and `runCatching` may swallow it and allow code to continue after cancellation.

```kotlin
try {
    work()
} catch (e: CancellationException) {
    throw e
} catch (e: Exception) {
    log(e)
}
```

When wrapping suspend calls, preserve cancellation explicitly:

```kotlin
suspend fun loadSafely(): Result<User> {
    return try {
        Result.success(repository.loadUser())
    } catch (e: CancellationException) {
        throw e
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

## finally and NonCancellable

`finally` always runs, but suspend calls inside it still respond to cancellation. Use `withContext(NonCancellable)` only for cleanup that must complete.

```kotlin
finally {
    withContext(NonCancellable) {
        flushAndClose()
    }
}
```

## Timeout and Resource Cleanup

Prefer `withTimeoutOrNull` when timeout is a normal empty result. When using `withTimeout`, catch `TimeoutCancellationException` explicitly.

Resources opened inside a timeout block must be cleaned in `finally`.

Blocking lower-level APIs may not respond to coroutine cancellation, even on `Dispatchers.IO`. When needed, combine coroutine cancellation with the SDK's own timeout, cancel, close, or interrupt mechanism.
