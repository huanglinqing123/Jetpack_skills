# Structured Concurrency

## Scope Lifetime

Do not use `GlobalScope` in production code. It is not owned by the caller's lifecycle. When a page exits, a ViewModel is cleared, or a business flow ends, `GlobalScope` work does not naturally cancel. The caller also cannot await completion or observe failures predictably.

Prefer:

- UI: `viewModelScope`, `lifecycleScope`, `rememberCoroutineScope`
- Parallel work inside suspend functions: `coroutineScope {}` or `supervisorScope {}`
- Long-lived background work: injected `CoroutineScope` with a documented owner and cancellation timing

```kotlin
suspend fun syncAll() = coroutineScope {
    launch { syncProfile() }
    launch { syncSettings() }
}
```

## launch and async

- Use `launch` for work that does not return a value.
- Use `async` only when a result is needed.
- Every `Deferred` must be awaited with `await()` or `awaitAll()`.
- An un-awaited `async` discards the result and exposes failures in the wrong place.

```kotlin
val page = coroutineScope {
    val main = async { loadMainData() }
    val extra = async { loadExtraData() }
    Page(main.await(), extra.await())
}
```

## awaitAll Exception Propagation

In `coroutineScope { async { ... }.awaitAll() }`, if any child fails, the parent scope is cancelled and sibling tasks are cancelled as well. This is correct for strongly dependent work, but wrong for independent modules.

For independent concurrent work, prefer `supervisorScope` and have each task produce a `Result` or fallback value explicitly.

```kotlin
val results = supervisorScope {
    loaders.map { loader ->
        async { runCatching { loader.load() } }
    }.awaitAll()
}
```

## Do Not Pass Job Directly to Builders

Do not write:

```kotlin
scope.launch(Job()) { work() }
scope.async(SupervisorJob()) { work() }
withContext(Job()) { work() }
```

This replaces the parent job in the coroutine context and breaks the parent-child relationship. Use `supervisorScope {}` for local supervision. For long-lived supervision, define `CoroutineScope(SupervisorJob() + dispatcher)` and make a clear owner cancel it.

## launch as the Last Line of coroutineScope

`suspend fun foo() = coroutineScope { launch { work() } }` looks like fire-and-forget, but `coroutineScope` waits for the child to finish. If work must continue in the background, use an external scope with a clear lifecycle and explain why.

## Do Not Use runBlocking in suspend Code

`runBlocking` blocks the current thread. Inside suspend functions or coroutine code, it defeats the non-blocking coroutine model and can cause Main-thread ANRs, thread-pool starvation, or slow tests.

Use `runTest` in tests and `coroutineScope` for child tasks.
