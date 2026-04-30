# State and Architecture

## Unidirectional Data Flow

Compose UI must follow state down, events up.

Do not pass ViewModel into ordinary reusable UI components or mutate business state directly from them.

Bad:

```kotlin
@Composable
fun UserName(viewModel: ProfileViewModel) {
    TextField(
        value = viewModel.name,
        onValueChange = { viewModel.name = it }
    )
}
```

Good:

```kotlin
@Composable
fun ProfileRoute(
    viewModel: ProfileViewModel = viewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    ProfileScreen(
        name = uiState.name,
        onNameChange = viewModel::onNameChange
    )
}

@Composable
fun ProfileScreen(
    name: String,
    onNameChange: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    TextField(
        value = name,
        onValueChange = onNameChange,
        modifier = modifier
    )
}
```

Rules:

- Route layer connects ViewModel, lifecycle state collection, and navigation.
- Screen/component layer receives only state and callbacks.
- Ordinary UI components must be previewable without ViewModel, NavController, Repository, or Android framework dependencies.

## State Hoisting

State belongs in the lowest common parent of all readers and writers. Do not automatically put every state in ViewModel, and do not hide shared state inside child components.

Bad:

```kotlin
@Composable
fun SearchBox() {
    var query by rememberSaveable { mutableStateOf("") }
    TextField(value = query, onValueChange = { query = it })
}
```

Good:

```kotlin
@Composable
fun SearchScreen() {
    var query by rememberSaveable { mutableStateOf("") }

    SearchBox(
        query = query,
        onQueryChange = { query = it }
    )

    ResultList(query = query)
}
```

Ownership table:

| State type | Owner |
| --- | --- |
| Local UI-only state used by one component | Component |
| UI state shared by sibling components | Lowest common parent |
| Business state or screen state | ViewModel |
| Process restoration or navigation argument state | ViewModel + SavedStateHandle |

## remember and rememberSaveable

Use the smallest persistence scope that satisfies the requirement.

| Scenario | Recommended API |
| --- | --- |
| Survive recomposition only | `remember` |
| Survive configuration change | `rememberSaveable` |
| Business state or process restoration | `ViewModel` + `SavedStateHandle` |

Bad:

```kotlin
var text by remember { mutableStateOf("") }
```

Good:

```kotlin
var text by rememberSaveable { mutableStateOf("") }
```

Use `rememberSaveable` only for values that can be saved by Bundle or a custom Saver.

## Observable State Collections

Do not use non-observable mutable collections as Compose state. Mutating `mutableListOf`, `mutableMapOf`, or `mutableSetOf` does not notify Compose.

Bad:

```kotlin
val items = remember { mutableListOf<String>() }

Button(onClick = { items.add("new") }) {
    Text("Add")
}
```

Good:

```kotlin
val items = remember { mutableStateListOf<String>() }

Button(onClick = { items.add("new") }) {
    Text("Add")
}
```

Also acceptable:

```kotlin
var items by remember { mutableStateOf(emptyList<String>()) }

Button(onClick = { items = items + "new" }) {
    Text("Add")
}
```

## Immutable UI State

UI state should be immutable and stable.

Bad:

```kotlin
data class ProfileUiState(
    val tags: MutableList<String>,
    val name: String
)
```

Good:

```kotlin
@Immutable
data class ProfileUiState(
    val name: String,
    val avatarUrl: String,
    val tags: List<String>
)
```

Rules:

- Do not expose `MutableList`, `MutableMap`, `MutableSet`, or mutable data holders from UI state.
- Prefer immutable values and one-way replacement.
- Consider persistent immutable collections when the project already uses them and stability matters.

## Navigation Boundaries

Do not pass `NavController` into reusable UI rows, cards, or screen content components.

Bad:

```kotlin
@Composable
fun ItemRow(navController: NavController, item: Item) {
    Row(
        Modifier.clickable {
            navController.navigate("detail/${item.id}")
        }
    ) {
        Text(item.title)
    }
}
```

Good:

```kotlin
@Composable
fun ItemRow(
    item: Item,
    onClick: (String) -> Unit
) {
    Row(Modifier.clickable { onClick(item.id) }) {
        Text(item.title)
    }
}
```

Route layer:

```kotlin
ItemRow(
    item = item,
    onClick = { id -> navController.navigate("detail/$id") }
)
```

Navigation belongs in Route or navigation graph integration code. UI components emit events.
