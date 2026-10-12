# ANRs, blocking, and contention

Use for unresponsive input, receiver/service timeouts, frozen frames, or blocked threads.

## Identify the deadline and owner

Read the ANR reason and timestamps before editing. Input dispatch, no-focused-window, broadcast, service, and job-related ANRs have different completion contracts. Timeouts depend on platform, component, and sometimes OEM behavior; there is no universal five-second rule. Startup can consume a background component's deadline too.

Follow the wait: blocked thread → lock owner, binder peer, executor queue, or resource. A background worker holding a lock can stall main. If main is runnable but rarely scheduled, inspect CPU contention/system load rather than adding more workers. See [Perfetto](android-kotlin-perfetto.md).

## Diagnostic patterns

| Evidence | Investigation / fix candidate |
| --- | --- |
| Main waiting on monitor/future/latch | Locate owner and work inside the critical section; remove blocking waits, shorten lock scope, publish prepared results |
| Long or repeated binder transactions | Attribute calls; hoist repeated system queries/listeners where semantics permit; honor thread-affinity requirements |
| Broadcast using `goAsync()` still times out | Verify worker starts promptly and every path finishes; `goAsync` does not remove the deadline |
| Receiver/service launch hits heavy initialization | Include process startup and executor queue delay in the investigation |
| No focused window | Inspect first-window creation and focus flags, not only expensive Kotlin methods |
| `nativePollOnce` / idle main in a late dump | Seek earlier trace/cluster evidence; do not rewrite the event loop from this snapshot alone |
| Main flushes pending preference writes | Inspect `SharedPreferences.apply()` traffic around lifecycle changes; batch or redesign persistence with durability preserved |

For asynchronous receivers, bound short work and finish its pending result in all terminal paths. Handle task-submission failure and cancellation as well as normal completion. Schedule durable long work using an appropriate background API instead of leaving a receiver pending. Do not move APIs requiring main to arbitrary threads.

## Validate

Reproduce the triggering event, cancellation, and queue saturation. Compare blocked time and operation completion, then monitor affected field ANR clusters. Fixing one local stall does not establish a fleet-wide ANR reduction. A suspending coroutine wait is different from a blocking thread wait; inspect the implementation.

Sources: [Google ANR diagnosis](https://developer.android.com/topic/performance/anrs/diagnose-and-fix-anrs), [SharedPreferences lifecycle blocking](https://developer.android.com/reference/android/content/SharedPreferences), [Compose binder example](https://developer.android.com/codelabs/jetpack-compose-performance).
