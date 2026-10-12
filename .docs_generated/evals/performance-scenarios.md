# Performance skill behavioral evaluation scenarios

These are evaluation inputs and reviewer criteria, not executable tests or completed model-evaluation results. Keep this file out of the skill's normal reference routing. Use it when comparing skill revisions.

## Procedure

Give an evaluator one input below and access to the candidate skill. Do not give it the criteria until its answer is recorded. Use the same model/settings, input artifacts and tool access for old/new revisions; use separate sessions. Evaluate the diagnosis and proposed validation, not exact phrasing or the number of suggestions. For mutation tests, supply a small isolated fixture and no live services. Code-only inputs cannot earn claims of measured speedup.

Score each applicable dimension 0–2: diagnosis, evidence/uncertainty, fix semantics, version/scope awareness, validation. Report raw answers, failures and sample sizes; do not infer performance gains in real apps from rubric scores. Repeated cases are needed before claiming a behavioral improvement.

## Inputs and reviewer criteria

| ID | Input to evaluator | Reviewer criteria / failure to catch |
| --- | --- | --- |
| E01 | `Application.onCreate` launches eight CPU-heavy initializers on Default. Launch janks on weak phones. Is it main-safe now? | Worker contention and initializer dependencies; bound/defer only optional work, compare TTID/TTFD; no “off main means fixed” |
| E02 | Input ANR snapshot ends in `nativePollOnce`, captured after the freeze. Fix the loop. | Insufficient attribution/late sample; seek trace/reason/other threads rather than rewrite Android's event loop |
| E03 | A receiver calls `goAsync`, submits to a saturated shared executor, and calls `finish` only after a successful upload. | Deadline remains, queue delay counts, failure/submission/cancellation cleanup, durable long work; no arbitrary timeout extension |
| E04 | Navigation occasionally hangs after thousands of small `SharedPreferences.apply()` calls. They are async, so can storage be excluded? | Lifecycle flush can block; inspect trace and write frequency; preserve persistence semantics |
| E05 | Twenty enter/back cycles retain Activities through registered callbacks. Replace all references with weak references? | Trace owner/retaining path, pair lifecycle cleanup, preserve active callbacks; no blanket weak references |
| E06 | Java heap is flat, but a media/map screen grows process memory and disappears. Enable `largeHeap`? | Native/graphics/resources plus exit/system-pressure evidence; no diagnosis from heap alone |
| E07 | SQL plan says `SCAN` for a 15-row settings table; a 200k-row filtered feed has no index. Add indexes everywhere? | Selectivity/data size, compare actual plans/time, index costs and migrations; no universal scan prohibition |
| E08 | Every token-expiry response launches refresh and immediately retries an order-creation request. Raise concurrency? | Coordinated refresh, bounded retry and order idempotency; validate concurrency/errors without duplicate writes |
| E09 | Each screen entry enqueues another periodic upload; background requests continue every few seconds. | Queue/work identity and radio cost; unique policy matching behavior, batching/freshness, completed-work validation |
| E10 | Adding many more Baseline Profile entries worsens cold startup and memory. Profiles should always help, right? | Compiled footprint/coverage and modes; compare artifacts, trim only with evidence; no promised universal percentage |
| E11 | R8 release crash occurs in reflected JSON parsing. Disable shrinking globally? | Diagnose effective keeps/dynamic entry points, focused rules and release tests; no blanket global disabling |
| E12 | WebView is warmed asynchronously then constructed immediately on main. The first screen still stalls. | Check version/overload and implicit initialization; immediate use may wait; compare startup versus memory tradeoff |
| E13 | A mixed RecyclerView/Compose implementation disposes composition on every detach and rebuilds all rows. | Pooling/version/lifecycle contracts, counts and retained state; no architecture rewrite as first fix |
| E14 | A feed has a tiny nested LazyRow, per-item receiver, and large resource placeholder; recomposition count looks low. | Investigate subcomposition, binder and decode with traces, not only stability annotations |
| E15 | Jank rate rises from 5/100 to 4/50 frames while session length drops. Did performance regress? | Absolute jank fell while percentage rose; inspect journey/session changes and comparable cohorts; neither automatic regression nor automatic win |
| E16 | A benchmark improves 2% in its best run; memory rises 40%; later runs throttle. Ship? | Variability/repeated comparable runs, memory guardrail, sustained behavior; no claim from best run or ignored tradeoff |
| E17 | Update two independent StateFlows using two awaited functions. Is the combined UI snapshot now atomic? | Ordered completion is not atomic observation; combined/versioned contract where needed |
| E18 | Apply `@Immutable` to a model wrapping an externally mutable list, and `remember` to every lambda. | Contract violation/stale UI risk and strong-skipping configuration; targeted fix, compiler plus runtime evidence |

## Review status for 2026-10-12

The author used these cases for a manual coverage review and added missing guidance to the linked topic files. No independent agent run or blinded before/after evaluation was performed. Package validation and installer smoke tests are reported separately in the change summary.
