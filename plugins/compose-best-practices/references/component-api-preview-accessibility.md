# Component API, Preview, and Accessibility

## Modifier API

Public composables must expose `modifier: Modifier = Modifier`.

Good:

```kotlin
@Composable
fun UserAvatar(
    imageUrl: String,
    modifier: Modifier = Modifier
) {
    Image(
        painter = rememberAsyncImagePainter(imageUrl),
        contentDescription = null,
        modifier = modifier.size(48.dp)
    )
}
```

Rules:

- The caller must be able to participate in layout.
- Put `modifier` near the end of required visual parameters, before optional callbacks only if that matches the local codebase style.
- Do not create hidden fixed layout behavior that cannot be overridden.

## Component Parameters

Pass only the data a UI component needs. Do not pass entire domain models when the component only reads a few fields.

Bad:

```kotlin
@Composable
fun Header(news: News) {
    Text(news.title)
    Text(news.subtitle)
}
```

Good:

```kotlin
@Composable
fun Header(
    title: String,
    subtitle: String
) {
    Text(title)
    Text(subtitle)
}
```

When parameter count becomes high, introduce an immutable UI model:

```kotlin
@Immutable
data class HeaderUiState(
    val title: String,
    val subtitle: String
)
```

Prefer UI models at screen boundaries. Keep domain models out of reusable leaf UI unless every field and behavior is intentionally part of that UI contract.

## Accessibility and Test Semantics

Clickable icons must have meaningful semantics.

Good:

```kotlin
IconButton(onClick = onBack) {
    Icon(
        imageVector = Icons.AutoMirrored.Filled.ArrowBack,
        contentDescription = "Back"
    )
}
```

Decorative icons should not expose semantics:

```kotlin
Icon(
    imageVector = Icons.Default.Star,
    contentDescription = null
)
```

Rules:

- Interactive elements need discoverable semantics.
- Decorative images/icons should use `contentDescription = null`.
- Add stable test tags only when they support UI tests and follow local project conventions.

## Preview Coverage

Core UI components should provide previews. Preview coverage is required for reusable components, complex state pages, empty states, loading states, error states, long text, dark mode, and multi-language-sensitive layouts.

Bad:

```kotlin
@Composable
fun DeviceCard(
    state: DeviceCardUiState,
    onClick: () -> Unit
) {
    // UI content
}
```

Good:

```kotlin
@Preview(showBackground = true)
@Composable
private fun DeviceCardPreview() {
    AppTheme {
        DeviceCard(
            state = DeviceCardUiState(
                name = "Amazfit Active 2",
                batteryText = "86%",
                isConnected = true
            ),
            onClick = {}
        )
    }
}
```

Add separate previews for important states:

```kotlin
@Preview(name = "Connected", showBackground = true)
@Composable
private fun DeviceCardConnectedPreview() {
    AppTheme {
        DeviceCard(
            state = DeviceCardUiState(
                name = "Amazfit Active 2",
                batteryText = "86%",
                isConnected = true
            ),
            onClick = {}
        )
    }
}

@Preview(name = "Disconnected", showBackground = true)
@Composable
private fun DeviceCardDisconnectedPreview() {
    AppTheme {
        DeviceCard(
            state = DeviceCardUiState(
                name = "Amazfit Active 2",
                batteryText = "--",
                isConnected = false
            ),
            onClick = {}
        )
    }
}
```

Preview rules:

- Preview ordinary UI components, not ViewModel-heavy Route layers.
- Include empty, loading, success, error, long text, and dark mode states when relevant.
- Keep preview data local, deterministic, and dependency-free.
