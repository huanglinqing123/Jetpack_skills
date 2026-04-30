# Dispatchers

## Main/UI Default Thread

`viewModelScope.launch {}` and `lifecycleScope.launch {}` run on Main by default. They are appropriate for updating UI state, reacting to lifecycle events, and calling main-safe suspend functions. They do not make every statement inside the coroutine safe to run on the UI thread.

Blocking I/O or CPU-heavy work must be moved:

- Switch inside the coroutine body with `withContext(ioDispatcher/defaultDispatcher)`.
- Or launch/async with the appropriate dispatcher.

## Dispatcher Choice

- UI state updates: `Dispatchers.Main` or `Main.immediate`
- Blocking I/O: `Dispatchers.IO`
- CPU-heavy work: `Dispatchers.Default`
- Production code should not use `Dispatchers.Unconfined` by default.

```kotlin
viewModelScope.launch {
    val data = withContext(ioDispatcher) {
        api.blockingLoad()
    }
    _state.value = UiState.Content(data)
}
```

## Main-Safe Suspend Functions

A suspend function does not automatically switch threads. If it performs blocking work internally, calling it from Main can still block the UI.

Repository and DataSource suspend APIs should be main-safe by switching to the right dispatcher internally.

```kotlin
class ConfigRepository(
    private val ioDispatcher: CoroutineDispatcher
) {
    suspend fun readConfig(): Config = withContext(ioDispatcher) {
        file.readText().decodeConfig()
    }
}
```

## Inject Dispatchers

Hardcoding `Dispatchers.IO`, `Dispatchers.Default`, or `Dispatchers.Main` makes tests harder to control and prevents virtual-time execution.

Recommendations:

- Inject `ioDispatcher` into Repository/DataSource classes.
- Inject `defaultDispatcher` into CPU-heavy classes.
- Use `Dispatchers.setMain(testDispatcher)` in ViewModel tests that touch Main.
