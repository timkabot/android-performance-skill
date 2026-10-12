# Storage, networking, and data volume

Use for slow data-backed interactions, repeated loads, growing offline datasets, or transfer/decoding cost.

## Storage

| Pattern | Confirm | Fix candidate |
| --- | --- | --- |
| Query degrades as data grows | Representative row counts and `EXPLAIN QUERY PLAN` | Selective/composite indexes matching filters/order; account for write/storage cost |
| N queries for N items | Query count per interaction | Fetch required relations/columns in batches; avoid loading an entire database |
| Bulk writes stall readers/UI | Transaction duration, contention, WAL/checkpoint behavior | Batch appropriately, shorten transactions; never hold a transaction across network work |
| Unnecessary full result materialization | Row/column counts and mapping allocations | Filter/aggregate in SQL when equivalent, paginate bounded results |
| Preference updates hitch on navigation | Pending writes and lifecycle trace | Reduce write frequency or use suitable async storage; preserve durability and migration semantics |

A full scan can be correct for a tiny table or low-selectivity query. Do not add indexes solely because a plan says `SCAN`. Measure cold/warm query time and writes. Room's asynchronous queries can still be expensive; moving a query off main does not repair its plan. Repeated invalidation may rerun observers even when the final result is equal.

## Networking

Separate request queueing, connection setup, server wait, body transfer, decode, mapping, and render. For OkHttp, use per-call `EventListener` state: reused connections omit setup events; retries/redirects can repeat events; failures can leave start/end pairs incomplete. Record only needed timings, not private headers or bodies.

| Pattern | Confirm | Fix candidate |
| --- | --- | --- |
| Every request reconnects | Connection reuse and client lifetime | Reuse appropriately configured clients/pools; close response bodies |
| Duplicate requests on navigation/rotation | Request identity and producer/collector count | Share or dedupe at the owning layer; preserve cancellation and freshness |
| Latency spikes after errors | Retry count, backoff, auth refresh concurrency | Bound retries, coordinate token refresh, honor server guidance and idempotency |
| Fast headers but slow usable content | Payload size, transfer/decode/map stages | Request fewer fields/items, stream where suitable, avoid duplicate parsing |
| Cache never helps or returns wrong data | Cache directives, identity, validation, user partitioning | Fix freshness/invalidation and reuse; never force caching across authorization boundaries |

Validate slow/offline/reconnect conditions and stale-response races, not only fast Wi-Fi. Compare latency, bytes, request count, failures, and freshness. Increasing all timeouts or dispatcher concurrency may hide symptoms while raising tail latency, server load, and memory. Do not retry writes such as payments without their established idempotency contract.

Sources: [SQLite performance](https://developer.android.com/topic/performance/sqlite-performance-best-practices), [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences), [OkHttp events](https://github.com/square/okhttp/blob/master/docs/features/events.md), [OkHttp caching](https://github.com/square/okhttp/blob/master/docs/features/caching.md), [network troubleshooting](https://developer.android.com/develop/connectivity/troubleshoot-network-issues).
