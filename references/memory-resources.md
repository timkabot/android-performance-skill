# Memory, images, and resource ownership

Use for OOM, repeated reloads, long-session degradation, GC-related jank, or retained screens/resources.

## Classify before changing

Distinguish allocation churn, retained live objects, intentional bounded caches, native/graphics memory, and system memory pressure. Compare the same journey after warm-up and repeated return to a comparable idle state. A rising peak alone does not prove a leak; a flat Java heap does not rule out native/graphics growth. Distinguish an OOM exception from an LMK termination or ordinary process recreation using available exit diagnostics.

## Diagnostic patterns

| Pattern | Evidence to collect | Fix candidate and tradeoff |
| --- | --- | --- |
| Screen survives repeated back navigation | Heap dominator/retaining path to Activity, View, callback, or listener | Correct owner scope and unregister/dispose; avoid blanket weak references |
| Memory plateaus only at a huge size | Cache entries/bytes, eviction and hit rate | Bound by bytes and workload; smaller cache can increase decode/network work |
| Small image file causes large spike | Decoded dimensions, bitmap format, concurrent requests, graphics/native usage | Request display-sized assets, bound concurrency, avoid repeated transformations |
| Scroll triggers frequent allocation/GC | Allocation stacks plus matching frame stalls | Remove repeated hot-path objects; pooling only if measured and ownership is safe |
| Memory rises with media/map/web screens | Native allocations, handles, active players/surfaces, renderer processes | Release resources at their real lifecycle boundary; respect thread contracts |
| Java heap is flat but process disappears | System pressure/LMK/exit information, PSS/RSS and native categories | Reduce total footprint and preserve state; do not assume `largeHeap` fixes it |
| Exception/cancellation leaks a resource | Acquire/release paths for cursors, streams, descriptors, listeners, wake locks | Use scoped cleanup / `finally` where valid, including partial initialization |

Decoded RGBA pixels commonly need roughly width × height × 4 bytes before extra buffers and renderer costs. Verify actual format/allocation; compressed file size is not decoded footprint. Do not change color depth or transparency just to reduce bytes without visual checks.

Leak repair must preserve legitimate use across lifecycle transitions. Garbage collection does not substitute for closing system resources. Avoid forced GC, indiscriminate cache eviction on every navigation, and application-wide retention of UI contexts.

## Validate

Repeat navigation, rotation/recreation, background/foreground, cancellation, and failed resource acquisition. Measure retained footprint after settling, peak footprint, allocation rate, and the cost of cache misses. Use heap dumps/allocation profiling for attribution, then confirm timing without their overhead. Test restoration separately from timing; forced process death is not a representative warm-start benchmark.

Sources: [manage app memory](https://developer.android.com/topic/performance/memory/manage-app-memory), [system memory and LMK](https://developer.android.com/topic/performance/memory/guide/system-wide-memory), [DroidLeaks](https://arxiv.org/abs/1611.08079), [PlumbDroid](https://arxiv.org/abs/2003.03201). The historical leak datasets motivate lifecycle/error-path checks; they do not measure current market prevalence.
