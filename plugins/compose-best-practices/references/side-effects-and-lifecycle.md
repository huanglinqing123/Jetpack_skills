# Side Effects and Lifecycle

## Side Effects

Composable bodies may recompose often. Never directly execute loading, navigation, Toast, listener registration, analytics, or external state synchronization in the composable body.

Bad:

```kotlin
@Composable
fun DetailScreen(id: String, repository: Repository) {
    repository.load(id)
}
```

Good:

```kotlin
@Composable
fun DetailScreen(
    id: String,
    viewModel: DetailViewModel
) {
    LaunchedEffect(id) {
        viewModel.load(id)
    }
}
```

Common effects:

| API | Use |
| --- | --- |
| `LaunchedEffect(key)` | Start coroutine when entering composition or key changes |
| `DisposableEffect(key)` | Register and clean up listeners |
| `SideEffect` | Publish successful recomposition state to non-Compose code |
| `rememberCoroutineScope` | Start UI-scoped coroutine from event callbacks |
| `rememberUpdatedState` | Keep latest lambda/value without restarting an effect |

## Complete Effect Keys

Treat an effect key as the lifecycle boundary of the work. If work depends on a value, that value must be in the key set.

Bad:

```kotlin
LaunchedEffect(Unit) {
    viewModel.load(userId)
}
```

Good:

```kotlin
LaunchedEffect(userId) {
    viewModel.load(userId)
}
```

If using a stable callback inside a long-lived effect, prefer `rememberUpdatedState` instead of adding the callback as a restart key.

## Do Not Write State During Composition

Do not write Compose state directly from the composable body. This can cause infinite recomposition or unstable UI behavior.

Bad:

```kotlin
@Composable
fun BadCounter() {
    var count by remember { mutableIntStateOf(0) }

    Text("$count")
    count++
}
```

Good:

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableIntStateOf(0) }

    Button(onClick = { count++ }) {
        Text("$count")
    }
}
```

State writes belong in user events, ViewModel intents, or explicit effects.

## Flow Collection

Use lifecycle-aware Flow collection in Compose screens.

Bad:

```kotlin
val uiState by viewModel.uiState.collectAsState()
```

Good:

```kotlin
val uiState by viewModel.uiState.collectAsStateWithLifecycle()
```

Rules:

- Prefer `collectAsStateWithLifecycle` for Android UI.
- Keep collection in Route or screen entry layers.
- Do not collect Flow separately in multiple children when one parent collection can provide derived UI state.
