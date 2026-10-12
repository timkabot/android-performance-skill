---
name: android-performance
description: "Diagnose and improve Android app performance: startup, ANRs, Compose and View rendering, memory, data loading, background battery use, and release optimization. Use for performance investigations, fixes, and reviews."
---

# Android Performance

Find the cause of slow interactions, resource waste, or lost responsiveness, then validate the user-visible result. Adapt to the target project's architecture, build commands, API versions, and instrumentation. No particular base ViewModel, state container, analytics SDK, or dispatcher wrapper is required.

## Workflow

1. Establish the symptom, affected journey, release, and device/OS cohort. For a broad audit, inspect available field metrics and release configuration, then rank suspected issues by user impact, exposure, and evidence. Continue with code inspection when telemetry is unavailable; do not invent frequency or savings.
2. Read the complete affected path: UI, state owner, mapper, data source, list items, and relevant effects. Check compiler/library versions and existing performance tooling before choosing APIs.
3. Trace the critical path from input through asynchronous work, state publication, composition, layout, draw, image decoding, and secondary effects. Find repeated work and observable intermediate states.
4. Read only the references relevant to the suspected cause:
   - [Startup](references/startup.md): TTID/TTFD, initializers, SDKs, navigation, and first-use work.
   - [ANRs and contention](references/anrs-contention.md): ANR type, binder, lock owners, receiver deadlines, and misleading stack dumps.
   - [Memory and resources](references/memory-resources.md): leaks, allocation churn, images, native memory, and process loss.
   - [Storage and networking](references/storage-network.md): SQL, persistence, duplicate requests, caching, retries, and payloads.
   - [Background work and power](references/background-power.md): jobs, wake locks, sensors, radio use, and sustained thermal cost.
   - [Release optimization](references/release-optimization.md): R8, Baseline/Startup Profiles, package footprint, and SDK costs.
   - [Views, graphics, and WebView](references/views-graphics-webview.md): layout, RecyclerView, interoperability, GPU cost, and web content.
   - [Main thread and coroutines](references/main-thread.md): expensive work, dispatcher ownership, cancellation, and Flow.
   - [Compose stability and recomposition](references/compose-stability-recomposition.md): model contracts, strong skipping, state reads, and effects.
   - [Lazy lists](references/lazy-lists.md): keys, nested layouts, item cost, pagination, and scroll restoration.
   - [Animations](references/animations.md): invalidated phases, visibility, and repeated animation work.
   - [State and critical paths](references/state-and-critical-paths.md): coordinated stores, initial rendering, images, and prefetch.
   - [Runtime and Perfetto](references/android-kotlin-perfetto.md): allocations, scheduling, trace analysis, and runtime optimization.
   - [Tracing and validation](references/tracing-validation.md): measurement design, compiler reports, frame metrics, and reporting.
5. Implement the smallest change that removes a supported cause. Preserve behavior, cancellation, accessibility, state restoration, and module ownership. An analysis-only request does not authorize edits.
6. Run the target project's relevant correctness checks. For runtime claims, compare the same journey before and after on representative physical hardware, including a lower-tier device when relevant.

## Match the evidence to the symptom

| Symptom | Start with | Avoid assuming |
| --- | --- | --- |
| Slow launch or first navigation | Startup trace, readiness definition, initializer graph | A fast splash means usable content is ready |
| ANR or frozen input | ANR reason, affected thread, lock/binder counterpart | The sampled top frame caused the stall |
| Scroll/animation jank | Frame timeline, main/render threads, item work | Every missed frame is a recomposition problem |
| App reloads or degrades after repeated use | Exit reason, memory trend, retained objects/resources | Every process death is a Java OOM |
| Slow data-backed interaction | Queueing, requests, SQL plan, decode/map/publish stages | Backend time equals client latency |
| Background drain or sustained slowdown | Work ownership, wakeups, power/thermal evidence | Less CPU time necessarily means less energy |
| Release-only regression | Optimized artifact, profiles, dependency/config diff | Debug timing represents the shipped app |

## Decision rules

- Do not add `remember`, `derivedStateOf`, `@Immutable`, persistent collections, dispatcher hops, or caching mechanically. Identify the work or invalidation they would avoid.
- Prefer eliminating repeated work before moving it to another thread. Background work still competes for CPU, memory, and scheduling.
- A `suspend` function is not inherently main-safe. Inspect the implementation and distinguish blocking APIs from APIs that already dispatch internally.
- Keep related visual state coherent. Several assignments may coalesce into one frame; several stores updated in sequence are not automatically an atomic transaction. Verify observation and suspension boundaries.
- Do not use Compose observation, item counts, or list size as a completion barrier between stores. Prefer explicit operation completion and a documented publication contract.
- Protect lower-tier devices. Fast hardware can hide scheduling, allocation, and decoding costs.
- Treat reduced CPU work, fewer recompositions, and compiler skippability as evidence about a mechanism, not proof of smoother rendering.
- State a falsifiable hypothesis and the expected trace/metric change before editing. If measurements contradict it, revisit the cause instead of stacking optimizations.
- Check the other side of a tradeoff: caching and prefetch can grow memory, parallel work can raise heat, and lazy initialization can shift latency to first interaction.
- Verify version-specific APIs, tooling, and Play metric definitions against current primary documentation. Historical talks and case-study percentages are leads, not universal rules or promised gains.

## Report

Explain the likely or measured cause, the change and why it addresses that cause, the validation performed, and remaining behavior or measurement risks. Distinguish a code-level hypothesis, a compiler result, and a measured runtime improvement. If no device or trace was available, give a concrete next measurement instead of claiming a performance win.

For each actionable finding, include the location, triggering condition, mechanism, proposed fix, and validation. Label evidence as **measured**, **code-supported**, or **hypothesis**. Use the [research review](.docs_generated/research/2026-10-12-android-performance-landscape.md) only when source rationale or broader coverage is needed; do not load the entire bibliography for a narrow fix.
