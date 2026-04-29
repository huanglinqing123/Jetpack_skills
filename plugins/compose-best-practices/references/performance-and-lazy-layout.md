# Performance and Lazy Layout

## Lazy Lists

Lazy lists must use stable keys when item identity matters.

Bad:

```kotlin
LazyColumn {
    items(messages) { message ->
        MessageRow(message)
    }
}
```

Good:

```kotlin
LazyColumn {
    items(
        items = messages,
        key = { it.id }
    ) { message ->
        MessageRow(message)
    }
}
```

For mixed item types, add `contentType`:

```kotlin
LazyColumn {
    items(
        items = elements,
        key = { it.id },
        contentType = { it.type }
    ) { element ->
        FeedItem(element)
    }
}
```

Missing keys are high severity when the list supports insert, delete, reorder, paging, animation, remembered item state, or expensive rows.

## Scroll Containers

Do not nest same-direction scroll containers without explicit size. A `LazyColumn` inside a vertically scrolling `Column` receives unbounded height and can lose lazy behavior or crash.

Bad:

```kotlin
Column(Modifier.verticalScroll(scrollState)) {
    Header()

    LazyColumn {
        items(items) { ItemRow(it) }
    }
}
```

Good:

```kotlin
LazyColumn {
    item { Header() }

    items(items, key = { it.id }) {
        ItemRow(it)
    }

    item { Footer() }
}
```

If nesting is unavoidable, constrain the child:

```kotlin
Column(Modifier.verticalScroll(scrollState)) {
    LazyColumn(
        modifier = Modifier.height(200.dp)
    ) {
        items(items, key = { it.id }) { ItemRow(it) }
    }
}
```

## Expensive Work

Avoid sorting, filtering, grouping, parsing, formatting large lists, or allocating heavy objects in composable bodies or Lazy item lambdas.

Bad:

```kotlin
LazyColumn {
    items(contacts.sortedWith(comparator)) {
        ContactRow(it)
    }
}
```

Good for UI-only lightweight derivation:

```kotlin
val sortedContacts = remember(contacts, comparator) {
    contacts.sortedWith(comparator)
}

LazyColumn {
    items(sortedContacts, key = { it.id }) {
        ContactRow(it)
    }
}
```

Prefer moving business sorting, filtering, aggregation, and mapping to ViewModel or UseCase. Use `remember` only for UI-local derived data.

## derivedStateOf

Use `derivedStateOf` only when inputs change more frequently than the UI needs to update.

Good:

```kotlin
val listState = rememberLazyListState()

val showTopButton by remember {
    derivedStateOf {
        listState.firstVisibleItemIndex > 0
    }
}
```

Bad:

```kotlin
val fullName by remember {
    derivedStateOf { "$firstName $lastName" }
}
```

Good:

```kotlin
val fullName = "$firstName $lastName"
```

`derivedStateOf` has overhead. Do not use it for ordinary string concatenation or simple value mapping.

## Defer High-Frequency Reads

Avoid reading high-frequency state too high in the composition tree. Pass lambdas or use lambda-based modifiers to delay reads to the smallest invalidation scope.

Bad:

```kotlin
Title(scroll = scrollState.value)
```

Better:

```kotlin
Title(scrollProvider = { scrollState.value })

@Composable
fun Title(scrollProvider: () -> Int) {
    val scroll = scrollProvider()
    Text("Scroll Offset: $scroll")
}
```

For drawing and layout reads, prefer APIs that read state in the phase that needs it, such as lambda overloads or draw/layout modifiers.
