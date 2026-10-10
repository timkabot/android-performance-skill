# Android runtime, Kotlin, and Perfetto

Use when frame misses may involve scheduling, allocations, decoding, locks, or rendering outside composition.

## Runtime causes

- Main-thread I/O, synchronous binder calls, lock contention, and waiting for worker results can delay input and rendering. An ANR threshold is not an acceptable frame budget; thresholds depend on the operation.
- `StrictMode` helps find accidental blocking I/O during development. It does not detect every performance problem.
- Repeated temporary allocations can cause GC pressure. Look for collection chains, formatting, regex construction, rich text, and bitmap churn in frequent paths.
- Sequences avoid some intermediate collections but introduce lazy iteration overhead. Measure before replacing simple loops or small collection operations.
- Primitive Compose state can avoid boxing for hot numeric values. Value classes can still box at generic/interface/nullable boundaries; do not assume allocation-free behavior.
- Large inlined helpers, reflection, broad keep rules, and object caches have size or memory tradeoffs. Avoid speculative micro-optimization.

## Trace workflow

1. Capture a repeatable interaction on representative physical hardware. Separate startup, loading, refresh, and scrolling so causes remain attributable.
2. Mark a few meaningful stages, for example `Feed.decode`, `Feed.map`, `Feed.publish`, and `Feed.prefetch`. Keep names stable and omit user data and per-item IDs.
3. Locate a missed frame and inspect the main thread, `RenderThread`, GPU-related work where available, decode workers, coroutine workers, binder, GC, and thread scheduling in the same window.
4. Distinguish executing CPU work from runnable-but-unscheduled time and blocked/sleeping time. Moving work off main cannot fix a GPU bottleneck or CPU starvation on its own.
5. Compare counts and duration distributions for the same stages. PerfettoSQL can join slices with process/thread information and expose waits or contention.
6. Correlate the trace with the measured interaction and frame timeline; a nearby long slice is not automatically the cause.

Use balanced, exception-safe trace scopes. Synchronous thread sections cannot span arbitrary suspension and thread migration. Use an appropriate asynchronous trace API or trace synchronous pieces separately. Keep instrumentation coarse enough that it does not become the workload.

## Release behavior

Use a release-like profileable build for runtime conclusions. Check shrinking/optimization and benchmark compilation mode. Baseline Profiles can help critical startup and interaction paths, but do not remove algorithmic work, blocking I/O, or excess layout. Startup Profiles affect code layout and have a different role. Preserve the target project's existing build setup unless changing it is part of the task.

Sources: [Perfetto Android trace analysis](https://perfetto.dev/docs/getting-started/android-trace-analysis), [Trace Processor](https://perfetto.dev/docs/analysis/trace-processor), [Android memory](https://developer.android.com/topic/performance/memory), [Baseline Profiles](https://developer.android.com/topic/performance/baselineprofiles/overview).
