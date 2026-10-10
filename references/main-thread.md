# Main thread and coroutines

Use when mapping, callbacks, blocking calls, or stream processing may delay rendering.

## Investigate ownership

Search the affected path for `viewModelScope.launch`, `Dispatchers.Main`, `withContext`, `flowOn`, collection transformations, formatting, parsing, bitmap work, synchronous storage calls, and effects. Follow callees: the launch site alone does not tell you where all work runs.

- Move substantial CPU work to a CPU dispatcher, normally an injected `Dispatchers.Default` equivalent.
- Move genuinely blocking I/O to an I/O dispatcher. Do not wrap every Room, Retrofit, or DataStore suspend call in another dispatcher: inspect whether the API already provides main-safety.
- Let the layer performing expensive work own its main-safety contract. Reuse existing injected dispatchers; do not introduce a dependency-injection framework for this skill.
- Respect each state container's threading contract. UI objects need main-thread access; `MutableStateFlow.update` itself is thread-safe. Keep expensive work out of update lambdas, which may be retried.
- Avoid hops around tiny calculations. A context switch can add queueing and introduce an observation boundary.

Illustrative mapping, with application-specific models and an injected `cpuDispatcher`:

```kotlin
viewModelScope.launch {
    val rows = withContext(cpuDispatcher) {
        response.items.map { mapper.toUiModel(it) }
    }
    state.update { it.copy(rows = rows, isLoading = false) }
}
```

This example does not handle competing requests by itself. Add cancellation or a request-generation check when refresh/search results can arrive out of order.

## Structured work

- Keep screen work in a lifecycle-owned scope. Use a longer-lived owner only for work explicitly intended to survive the screen.
- Bound concurrent requests and mapping. One `async` per item can starve rendering or image decoding even when nothing blocks main.
- Long CPU loops need cooperative cancellation, for example `ensureActive()` at sensible intervals.
- Use latest-result semantics only where intermediate results may safely be discarded. Do not cancel non-idempotent writes merely to reduce UI work.
- A suspending `Job.join()` does not block a thread; `runBlocking`, monitor waits, and blocking future reads can. Diagnose the actual operation.

## Flow

- `flowOn` affects upstream operators. Verify which mapping runs before and after it.
- Share expensive cold producers deliberately with `stateIn` or `shareIn` when multiple collectors should reuse one result. Choose lifecycle and replay behavior to preserve freshness.
- Reduce scroll observations to the smallest useful value, such as a pagination threshold. Apply `distinctUntilChanged` after that transformation; `StateFlow` already suppresses equal values.
- Use buffering, conflation, debounce, or latest collection according to event semantics. Buffering can increase stale work and latency.
- Inspect `callbackFlow` capacity and `trySend` failures under bursty input.
- Collect UI state with lifecycle-aware APIs where appropriate. Composition lifetime alone does not mean an item is visible or the screen resumed.

Source: [Android coroutine best practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices), [Kotlin Flow](https://kotlinlang.org/docs/flow.html).
