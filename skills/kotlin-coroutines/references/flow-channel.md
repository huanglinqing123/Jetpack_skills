# Flow and Channel

## Blocking Calls in Flow

`flow {}` does not automatically switch threads. A blocking call inside `flow {}` blocks the collector context.

```kotlin
fun observeData(): Flow<Data> = flow {
    emit(blockingLoad())
}.flowOn(ioDispatcher)
```

## Cold and Hot Streams

- `Flow`: cold stream. Every collection restarts upstream work. Use it for one-shot requests, queries, and transformation pipelines.
- `StateFlow`: hot stream with a current value. Use it for UI state.
- `SharedFlow`: hot stream for events or shared emissions. Design `replay`, `extraBufferCapacity`, and `onBufferOverflow` explicitly.

Use `StateFlow` for UI state, configured `SharedFlow` for one-shot events, `shareIn` for shared upstream work, and `stateIn` for state sharing.

## collectLatest

`collectLatest` cancels the previous item processing block when a new item arrives.

Good use cases:

- Search input.
- Rendering only the newest state.
- Old work becoming irrelevant when new input arrives.

Avoid it when:

- Every item must complete processing.
- The work writes to a database, uploads data, sends analytics, or must not be dropped.

## Lifecycle-Aware Collection

Direct `launch { flow.collect {} }` in Activity/Fragment may keep collecting when UI is not visible.

Rules:

- Use `viewLifecycleOwner.lifecycleScope` in Fragment.
- Use `repeatOnLifecycle(Lifecycle.State.STARTED)` or `flowWithLifecycle` for UI collection.
- Prefer `collectAsStateWithLifecycle()` in Compose.

```kotlin
viewLifecycleOwner.lifecycleScope.launch {
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect(::render)
    }
}
```

## SharedFlow Configuration

```kotlin
private val _events = MutableSharedFlow<UiEvent>(
    replay = 0,
    extraBufferCapacity = 1,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)
```

If the data represents the latest state, use `StateFlow` instead of `SharedFlow`.

## Channel

When manually creating a `Channel`, define its owner and close timing. Otherwise consumers may suspend forever or resources may leak.

- Prefer `produce {}` for single producers.
- Prefer `callbackFlow` for converting callbacks to Flow, and unregister callbacks in `awaitClose`.
- Do not share `consumeEach` across multiple consumers.
- Prefer `SharedFlow` for broadcast-style data.
