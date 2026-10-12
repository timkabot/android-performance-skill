# Startup and first interaction

Use for launch delays, blank/splash screens, slow deep links, or latency shifted to first use.

## Establish the journey

Separate cold, warm, and hot starts; first install/update; returning authenticated users; notification/deep-link entry; and background component starts. Trace from process creation through the first frame and usable content. TTID measures initial display; TTFD depends on a correct fully-drawn signal. Define readiness around the primary user task, not a spinner, network completion, or every optional widget.

## Diagnostic patterns

| Signal or code pattern | Confirm | Focused intervention |
| --- | --- | --- |
| Long application creation | Trace initializer/DI/static initialization and inspect merged manifest providers | Remove unnecessary eager work; preserve dependency ordering and SDK contracts |
| Many startup SDKs | Attribute time, threads, I/O, and provider initialization per SDK | Defer optional initialization or configure it on demand; check consent and first-use behavior |
| Loading animation competes with initial content | Compare scheduling/frame traces with the animation disabled in a test build | Simplify the loading UI only if measured and acceptable to UX |
| Low TTID but long wait to interact | Verify `reportFullyDrawn`/`FullyDrawnReporter` placement and actual input readiness | Include required asynchronous content in readiness; report optional work separately |
| Fast launch, slow first tap | Trace lazy dependency creation, database open, WebView, and profile coverage | Move only proven critical work earlier or amortize it without hurting launch |
| First install/update is worse | Check compilation/profile installation and migration time | Validate shipped profiles and migrations on those paths |

Main-safety is not enough: work launched on several workers during startup can compete with rendering. Model dependencies, limit concurrency, and measure the post-launch tail. Check whether a shared initializer also runs in secondary processes or background receiver/service launches.

## Validate

Use startup Macrobenchmarks for the actual entry points; keep build, compilation mode, data, and cache conditions comparable. Compare TTID and TTFD together with first interaction latency. Exercise failed/offline loading so the fully-drawn contract does not wait forever. Do not add a fixed splash delay to improve a reported metric. See [release optimization](release-optimization.md) for profiles.

Sources: [startup analysis](https://developer.android.com/topic/performance/appstartup/analysis-optimization), [profile measurement and TTFD](https://developer.android.com/topic/performance/baselineprofiles/measure-baselineprofile), [Gmail Wear OS case study](https://developer.android.com/topic/performance/appstartup/case-study-gmail-wear). The Wear OS result is an example of CPU contention, not a requirement to remove loading animations from phones.
