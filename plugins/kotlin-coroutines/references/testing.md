# Coroutine Testing

## Use runTest

Use `runTest` for coroutine unit tests. Do not use `runBlocking`. Avoid real `delay()` and `Thread.sleep()`; advance virtual time instead.

```kotlin
@Test
fun testDelay() = runTest {
    val job = launch { delay(1_000) }
    advanceTimeBy(1_000)
    job.join()
}
```

Common tools:

- `advanceTimeBy(...)`
- `runCurrent()`
- `advanceUntilIdle()`
- `StandardTestDispatcher`

## Replace Main Dispatcher

ViewModel, `viewModelScope`, and Main dispatcher code need Main replacement in JVM unit tests.

```kotlin
private val testDispatcher = StandardTestDispatcher()

@Before
fun setUp() {
    Dispatchers.setMain(testDispatcher)
}

@After
fun tearDown() {
    Dispatchers.resetMain()
}
```

## Awaitable Background Work

Ownerless fire-and-forget production work makes tests unable to wait for completion or assert state deterministically.

Rules:

- Inject scopes or dispatchers so tests can control execution.
- When using `viewModelScope`, replace Main dispatcher and call `advanceUntilIdle()`.
- Avoid creating uncontrolled `CoroutineScope(...)` inside code under test.

```kotlin
viewModel.load()
advanceUntilIdle()
viewModel.uiState.value shouldBe expected
```
