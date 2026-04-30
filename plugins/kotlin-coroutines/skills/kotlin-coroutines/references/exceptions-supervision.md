# Exceptions and Supervision

## Job vs SupervisorJob

Regular `Job` semantics:

- Child failures affect the parent.
- Parent cancellation cancels all children.
- One child failure usually cancels sibling jobs.

Use this for strongly dependent work that should succeed or fail as one unit, such as permission validation plus main data loading or transactional flows.

`SupervisorJob` semantics:

- Parent cancellation still cancels all children.
- A child failure does not automatically cancel sibling jobs.

Use this for relatively independent sections such as page cards, banners, recommendations, or optional modules.

The decision is not about which API is more advanced. Ask whether the child tasks are one business unit or independent units.

## supervisorScope vs SupervisorJob

- `SupervisorJob`: defines supervision semantics for a scope with a clear lifecycle owner.
- `supervisorScope`: applies supervision temporarily inside a structured concurrency block.

```kotlin
suspend fun loadIndependentSections() = supervisorScope {
    launch { loadBanner() }
    launch { loadFeed() }
}
```

Do not pass `SupervisorJob()` directly to a single builder such as `launch(SupervisorJob()) {}`. This usually does not provide the intended supervision and may sever the parent-child relationship.

## CoroutineExceptionHandler and async

`CoroutineExceptionHandler` handles uncaught coroutine exceptions. Exceptions from `async` are stored in `Deferred` and are normally thrown from `await()`.

- Use a handler at top-level `launch` boundaries to record uncaught failures.
- Always `await()` `async` results and handle errors at the await site.
- Do not expect a handler to catch an un-awaited `async` failure.

## Business Errors Are Not CancellationException

`CancellationException` represents coroutine cancellation. Do not subclass or throw it for business failures. Use ordinary exceptions, sealed result types, or domain errors.
