# Tracing and validation

Use for measurement design and for deciding what a performance claim can support.

## Before and after

Record build variant, device/OS, refresh rate, compilation/profile mode, data set, cache state, thermal conditions, and interaction. Keep conditions comparable, repeat runs, and distinguish cold/warm startup and first/subsequent visits. Emulators are useful for correctness and investigation, not representative end-user performance numbers.

At 60/90/120 Hz, display intervals are roughly 16.7/11.1/8.3 ms. Actual app deadlines depend on the rendering pipeline. Android vitals describes frames over 700 ms as frozen; check whether the chosen collection mechanism covers the rendering technology and journey being measured.

Prefer existing screen/window instrumentation. JankStats can associate frame data with UI state; avoid adding an independent frame listener per list item. Define data-ready consistently and keep labels low-cardinality.

## Choose evidence

| Question | Useful evidence |
| --- | --- |
| Did behavior survive? | Focused unit/UI tests and the affected module's build checks |
| Why does this subtree recompose? | Compiler reports, parameter identity, and recomposition tooling |
| What delayed a frame? | System trace / Perfetto with frame and thread timelines |
| Did a repeatable journey improve? | Macrobenchmark with comparable builds and repeated physical-device runs |
| Does the field regression affect weak devices? | Device-tier cohorts, absolute counts, and interaction/session distributions |

Macrobenchmark `FrameTimingMetric` provides frame timing distributions; metric availability, including frame overrun, depends on Android version. `TraceSectionMetric` can measure selected custom stages. Use startup-specific metrics for startup questions.

## Ratio traps

Check the exact definition of a metric named hitch rate or jank rate. A percentage can increase because total recorded frames fell, even when the number of slow frames did not rise. Compare numerator, denominator, frozen frames, session length, frame percentiles, and device mix. Do not multiply unrelated aggregate averages and present the result as a measured count.

## Static checks and reports

Discover commands from the target repository's instructions and Gradle configuration. Run the narrowest relevant checks; do not assume custom build or lint tasks exist. Compiler report locations and options are configurable: inspect the Compose compiler plugin setup before locating classes/composables reports or adding temporary reports.

Build success establishes no runtime improvement. Skippability is a compiler property, recomposition counts describe activity, and a trace explains a particular run. Report those separately from a repeated before/after benchmark result.

Sources: [rendering and Android vitals](https://developer.android.com/topic/performance/vitals/render), [JankStats](https://developer.android.com/topic/performance/jankstats), [Macrobenchmark metrics](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-metrics), [Macrobenchmark setup](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview).
