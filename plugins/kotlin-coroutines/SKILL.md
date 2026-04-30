---
name: kotlin-coroutines
description: Use when the user asks about Kotlin Coroutines, structured concurrency, coroutine scope ownership, Dispatchers, suspend functions, Flow, StateFlow, SharedFlow, Channel, cancellation, exception handling, coroutine testing, runTest, TestDispatcher, Android lifecycle-aware collection, viewModelScope, lifecycleScope, or needs a safety, correctness, or testability review for asynchronous Kotlin code.
version: 1.0.0
---

# Kotlin Coroutines Guide

Use this skill to design, review, or refactor Kotlin coroutine code so it is structured, cancellation-safe, main-safe, lifecycle-aware, and testable.

This skill applies to Android/Kotlin production code and unit tests that use `kotlinx.coroutines`, `Flow`, `StateFlow`, `SharedFlow`, `Channel`, `viewModelScope`, `lifecycleScope`, and `kotlinx-coroutines-test`.

## Agent Behavior

When this skill is active:

1. Identify the coroutine topic and risk first: scope lifetime, Dispatcher choice, cancellation behavior, exception propagation, Flow collection, Channel ownership, or test determinism.
2. Load the relevant reference before giving detailed guidance:
   - `references/structured-concurrency.md`
   - `references/dispatchers.md`
   - `references/cancellation.md`
   - `references/exceptions-supervision.md`
   - `references/flow-channel.md`
   - `references/testing.md`
3. For code review or refactoring requests, answer with:
   - Problem analysis
   - Problematic code
   - Improved code
   - Explanation
4. If the user asks a conceptual question without code, avoid forced code snippets unless an example materially improves clarity.
5. Do not suggest or preserve production code that violates the Hard Rules below.
6. If several coroutine issues exist together, provide one coherent fix instead of isolated micro-fixes.
7. Follow the project architecture, dependency injection style, and testing style already present in the codebase. In MAD Android projects, prefer Koin injection when providing `CoroutineDispatcher` or `CoroutineScope`.

## Quick Triage

| Symptom or Question | First Rule To Apply |
| --- | --- |
| `GlobalScope`, unclear lifetime, or "where should I launch?" | Use lifecycle-bound, injected, or local structured scopes |
| `async` without `await()` | Use `launch`, or ensure every `Deferred` is awaited |
| A suspend function launches work into an external scope | Preserve structured concurrency unless the work must outlive the caller |
| `runBlocking` inside suspend/coroutine code | Replace with suspend calls, `coroutineScope`, or `runTest` in tests |
| Blocking I/O on Main or Default | Move it to `withContext(ioDispatcher)` |
| Hardcoded `Dispatchers.IO/Default/Main` | Inject dispatchers for testability |
| Passing `Job()` or `SupervisorJob()` directly to `launch/async/withContext` | Use `coroutineScope`, `supervisorScope`, or a real owner scope |
| `catch (Exception)` around suspend calls | Rethrow `CancellationException` |
| Polling or infinite loops | Use `while (isActive)` and cancellable `delay` |
| `withTimeout` unexpectedly cancels parent work | Prefer `withTimeoutOrNull`, or catch `TimeoutCancellationException` explicitly |
| Expecting `CoroutineExceptionHandler` to catch `async` failures | Await the `Deferred`; handlers only observe uncaught `launch` failures |
| Slow or flaky coroutine tests | Use `runTest`, virtual time, and `TestDispatcher` |
| Permanent Flow collection in UI | Use `repeatOnLifecycle`, `flowWithLifecycle`, or `collectAsStateWithLifecycle` |
| `MutableSharedFlow` drops events or has unclear backpressure | Configure `replay`, `extraBufferCapacity`, and `onBufferOverflow` deliberately |

## Hard Rules

### Scope and Structured Concurrency

- Do not use `GlobalScope` in production code.
- Use framework-provided scopes for UI work:
  - `viewModelScope` in ViewModels
  - `lifecycleScope` in Activity/Fragment or other lifecycle owners
  - `rememberCoroutineScope` for Compose event handlers
- `viewModelScope.launch {}` and `lifecycleScope.launch {}` run on Main by default. Do not perform blocking I/O or heavy CPU work directly inside them. Switch with `withContext(ioDispatcher/defaultDispatcher)` or launch with the appropriate dispatcher.
- Use `coroutineScope {}` when a suspend function needs child tasks that must complete or cancel with the caller.
- Use an injected external `CoroutineScope` only when the work must outlive the caller. Document the owner and lifetime of that scope.
- Do not implement background fire-and-forget by ending a suspend function with `coroutineScope { launch { ... } }`; `coroutineScope` waits for all children.
- Do not pass `Job()` or `SupervisorJob()` directly to builders such as `launch(Job())`, `async(SupervisorJob())`, or `withContext(Job())`. Use `coroutineScope`, `supervisorScope`, or a clearly owned `CoroutineScope`.

```kotlin
// Bad: breaks structured concurrency and leaks lifetime ownership.
suspend fun sync() {
    GlobalScope.launch { repository.sync() }
}

// Good: the caller owns completion and cancellation.
suspend fun sync() = coroutineScope {
    launch { repository.sync() }
}
```

### launch and async

- Use `launch` for work that does not return a value.
- Use `async` only when a result is needed.
- Every `Deferred` must be awaited with `await()` or `awaitAll()`.
- Wrap parallel child work in `coroutineScope {}` or `supervisorScope {}`.

```kotlin
suspend fun loadDashboard(): Dashboard = coroutineScope {
    val profile = async { profileRepository.load() }
    val summary = async { summaryRepository.load() }

    Dashboard(
        profile = profile.await(),
        summary = summary.await()
    )
}
```

Use `supervisorScope` when sibling tasks are independent and one failure should not cancel the others.

```kotlin
suspend fun loadCards(): List<CardResult> = supervisorScope {
    cardLoaders
        .map { loader -> async { runCatching { loader.load() } } }
        .awaitAll()
}
```

### Dispatchers and Main-Safe Suspend Functions

- Use `Dispatchers.Main` or `Main.immediate` for UI-only work.
- Use `Dispatchers.IO` for blocking file, database, SDK, or network client calls.
- Use `Dispatchers.Default` for CPU-heavy parsing, sorting, compression, or computation.
- Do not use `Dispatchers.Unconfined` in production unless there is a rare and documented reason.
- Suspend functions must be main-safe: callers should be able to invoke them from Main without blocking UI.
- Prefer injected dispatchers over hardcoded dispatchers.

```kotlin
class UserRepository(
    private val api: UserApi,
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun loadUser(id: String): User = withContext(ioDispatcher) {
        api.blockingLoadUser(id)
    }
}
```

### Cancellation

- Do not swallow `CancellationException`.
- Do not use `CancellationException` for business errors.
- Call `yield()` or `ensureActive()` in long CPU loops.
- Use `while (isActive)` plus cancellable `delay(interval)` for polling.
- If cleanup in `finally` must call suspend APIs, wrap only that cleanup in `withContext(NonCancellable)`.
- Do not reuse a scope after `scope.cancel()`. Use `scope.coroutineContext.job.cancelChildren()` if the intent is only to stop child jobs.

```kotlin
try {
    repository.sync()
} catch (e: CancellationException) {
    throw e
} catch (e: Exception) {
    logger.e(e, "Sync failed")
}
```

```kotlin
fun CoroutineScope.startPolling(intervalMs: Long): Job = launch {
    while (isActive) {
        refresh()
        delay(intervalMs)
    }
}
```

### Timeout

- Prefer `withTimeoutOrNull` when timeout is an expected empty result.
- Catch `TimeoutCancellationException` explicitly when using `withTimeout`.
- Clean resources opened inside timeout blocks in `finally`.

```kotlin
val result = withTimeoutOrNull(5_000) {
    repository.awaitResult()
}
```

### Exceptions

- Uncaught exceptions in `launch` propagate to the parent and can be observed by a `CoroutineExceptionHandler` installed at a top-level boundary.
- Exceptions in `async` are stored in `Deferred` and are thrown from `await()`.
- `CoroutineExceptionHandler` is not a replacement for local error handling.
- Prefer explicit result types or ordinary exceptions for business failures.

```kotlin
private val handler = CoroutineExceptionHandler { _, throwable ->
    logger.e(throwable, "Unhandled coroutine failure")
}

fun refresh() {
    viewModelScope.launch(handler) {
        _state.value = UiState.Content(repository.load())
    }
}
```

## Flow Rules

### Flow Builders

- Keep `flow {}` non-blocking.
- Use suspend APIs inside `flow {}`.
- If blocking work is unavoidable, use `flowOn(ioDispatcher)` or move the blocking section to `withContext(ioDispatcher)`.

```kotlin
fun observeUsers(): Flow<List<User>> = flow {
    emit(api.blockingLoadUsers())
}.flowOn(ioDispatcher)
```

### Cold and Hot Streams

| Type | Use Case | Key Rule |
| --- | --- | --- |
| `Flow` | Cold stream or single collection pipeline | Each collector restarts upstream work |
| `StateFlow` | UI state with a latest value | Always has a current value |
| `SharedFlow` | Events or shared emissions | Configure replay and buffering explicitly |

Use `stateIn` for UI state and `shareIn` for shared upstream work. Prefer `SharingStarted.WhileSubscribed(...)` for lifecycle-aware UI data streams unless the app explicitly needs eager background work.

### collect and collectLatest

- Use `collect` when every item must finish processing.
- Use `collectLatest` only when the previous in-flight work should be cancelled when a newer item arrives, such as search input or latest-only rendering.

### Android Lifecycle Collection

UI Flow collection must be lifecycle-aware.

```kotlin
viewLifecycleOwner.lifecycleScope.launch {
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state ->
            render(state)
        }
    }
}
```

In Compose, prefer the project lifecycle-aware state collection API, such as `collectAsStateWithLifecycle()`.

### SharedFlow Events

- Configure `MutableSharedFlow` explicitly.
- One-shot events usually use `replay = 0`, unless late collectors must receive the most recent event.
- If emitters must not suspend, configure `extraBufferCapacity` and `onBufferOverflow`.

```kotlin
private val _events = MutableSharedFlow<UiEvent>(
    replay = 0,
    extraBufferCapacity = 1,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)
val events: SharedFlow<UiEvent> = _events
```

## Channel Rules

- Prefer `produce {}` for single producers because the channel closes automatically when the coroutine completes.
- When manually creating a `Channel`, define the owner and close timing.
- Do not share `consumeEach` across multiple consumers. Use `for (item in channel)` when multiple consumers are required by the design.

```kotlin
fun CoroutineScope.eventsFromSdk(sdk: Sdk): ReceiveChannel<Event> = produce {
    val listener = object : SdkListener {
        override fun onEvent(event: Event) {
            trySend(event)
        }
    }

    sdk.addListener(listener)
    awaitClose { sdk.removeListener(listener) }
}
```

## Testing Rules

- Use `kotlinx-coroutines-test`.
- Use `runTest` for coroutine unit tests; do not use `runBlocking`.
- Use virtual time: `advanceTimeBy(...)`, `runCurrent()`, and `advanceUntilIdle()`.
- Inject `TestDispatcher` into production classes under test.
- Replace Main dispatcher when tests touch `Dispatchers.Main` or `viewModelScope`.
- Avoid real `delay()` and `Thread.sleep()` in tests.

```kotlin
class UserViewModelTest {
    private val testDispatcher = StandardTestDispatcher()

    @Before
    fun setUp() {
        Dispatchers.setMain(testDispatcher)
    }

    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun `loads user`() = runTest(testDispatcher) {
        val viewModel = UserViewModel(repository, testDispatcher)

        viewModel.loadUser("1")
        advanceUntilIdle()

        viewModel.uiState.value shouldBe UiState.Content(user)
    }
}
```

If the project already depends on Turbine, prefer Turbine for Flow tests.

```kotlin
repository.observeUsers().test {
    awaitItem() shouldBe expectedUsers
    cancelAndIgnoreRemainingEvents()
}
```

## Common Refactor Patterns

### Replace Hardcoded Dispatchers

```kotlin
// Before
class Parser {
    suspend fun parse(raw: String) = withContext(Dispatchers.Default) {
        parseLargePayload(raw)
    }
}

// After
class Parser(
    private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    suspend fun parse(raw: String) = withContext(defaultDispatcher) {
        parseLargePayload(raw)
    }
}
```

### Replace fire-and-forget async

```kotlin
// Before
viewModelScope.async {
    analytics.trackClick()
}

// After
viewModelScope.launch {
    analytics.trackClick()
}
```

### Keep Repositories Main-Safe

```kotlin
class FileRepository(
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun readConfig(file: File): Config = withContext(ioDispatcher) {
        file.inputStream().use { stream ->
            decodeConfig(stream)
        }
    }
}
```

### Preserve Cancellation with runCatching

`runCatching` catches `CancellationException`; rethrow cancellation when wrapping suspend calls.

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

## Review Checklist

- [ ] Scope owner is clear and lifecycle-bound.
- [ ] Suspend functions do not start ownerless background work.
- [ ] Every `async` result is awaited.
- [ ] Blocking work runs on IO, CPU work runs on Default, and UI work runs on Main.
- [ ] Dispatchers that affect testability are injected.
- [ ] `CancellationException` is rethrown.
- [ ] Long loops and polling respond to cancellation.
- [ ] Timeout handling does not unexpectedly cancel parent work.
- [ ] Android UI Flow collection is lifecycle-aware.
- [ ] UI state uses `StateFlow`; events use `SharedFlow`.
- [ ] SharedFlow buffer and replay settings are deliberate.
- [ ] Coroutine tests use `runTest`, virtual time, and `TestDispatcher`.
