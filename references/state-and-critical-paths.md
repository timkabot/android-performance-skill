# State publication and the visible critical path

Use when network completion is fast but the first useful frame or refresh is slow, or when several stores contribute to one screen.

## Coherent publication

Map the path from request through decoding, domain/UI mapping, store updates, composition, layout, and first useful draw. Count observable intermediate states and trace their actual cost.

For related fields, prefer preparing a coherent result and publishing it together. A loading flag, header, rows, and refresh indicator can otherwise expose partial or impossible combinations. This is a consistency rule, not a blanket ban on progressive rendering: independent useful content may reasonably appear early.

Avoid successful refresh sequences that empty a visible store, launch asynchronous repopulation, wait for a count, and finally publish list rows. A count can match an old request, fail when items are deduplicated, or tell nothing about which items were published.

Use an explicit suspend completion contract when one store depends on another. Be precise about what completion means: mapping finished, update enqueued, or state published. An awaited update followed by another store update provides ordering, **not necessarily atomic observation**. If consistency across both stores is required, consider a single combined state owner, a suitable snapshot transaction for snapshot state, or versioned publication with consumers rejecting mixed versions. Do not put suspending work inside a snapshot mutation block or assume it makes unrelated Flow emissions atomic.

Multiple synchronous writes may coalesce before a frame. Do not claim one frame per assignment without a trace.

## Secondary work

- Decide which content makes the first viewport useful. Include dependent data in its readiness definition.
- Schedule noncritical widgets, popup checks, impression processing, and speculative work so they do not compete with the first useful draw. Publishing state alone does not mean that draw has completed.
- Use explicit readiness/visibility signals when ordering matters. A fixed delay is a heuristic, not a guarantee.
- Cancel or supersede stale work after refresh, query/filter changes, tab switches, or navigation.
- Build lookup indexes when source data changes if item rendering otherwise performs repeated linear scans. Weigh index memory and rebuild cost.

## Images and prefetch

Constrain decode dimensions to display needs. Keep cache/request identity compatible between prefetch and display, including size and transformations. A small compressed file may decode into a large bitmap.

Prefetch only when the next use is plausible, parameters are known, results remain fresh, and concurrency/memory budgets are bounded. Reduce speculative work on constrained devices; cancel stale requests and measure cache reuse. Do not require a particular image library or reduce bitmap quality without checking visual requirements.

Client request duration can include queueing, DNS, connection/TLS setup, transfer, decoding, and mapping. Do not label it backend processing time. Measure elapsed durations with a monotonic clock; use wall-clock timestamps only for calendar events.

Source: [Android UI state production](https://developer.android.com/topic/architecture/ui-layer/state-production), [bitmap loading](https://developer.android.com/topic/performance/graphics/load-bitmap).
