---
name: android-performance
description: Investigate and improve Android and Jetpack Compose performance, including jank, slow rendering, recomposition, heavy lists, coroutine work, allocation churn, and performance-sensitive code review.
---

# Android Performance

Find the work that delays a useful frame, remove its cause, and validate the user-visible result. Adapt to the target project's architecture, dependency injection, build commands, and instrumentation. No particular base ViewModel, state container, analytics SDK, or dispatcher wrapper is required.

## Workflow

1. Establish the symptom and interaction: startup, navigation, loading, refresh, scrolling, or animation. Record the affected device tier and available evidence. Continue with code inspection when a trace is unavailable; label hypotheses accordingly.
2. Read the complete affected path: UI, state owner, mapper, data source, list items, and relevant effects. Check compiler/library versions and existing performance tooling before choosing APIs.
3. Trace the critical path from input through asynchronous work, state publication, composition, layout, draw, image decoding, and secondary effects. Find repeated work and observable intermediate states.
4. Read only the references relevant to the suspected cause:
   - [Main thread and coroutines](references/main-thread.md): expensive work, dispatcher ownership, cancellation, and Flow.
   - [Compose stability and recomposition](references/compose-stability-recomposition.md): model contracts, strong skipping, state reads, and effects.
   - [Lazy lists](references/lazy-lists.md): keys, nested layouts, item cost, pagination, and scroll restoration.
   - [Animations](references/animations.md): invalidated phases, visibility, and repeated animation work.
   - [State and critical paths](references/state-and-critical-paths.md): coordinated stores, initial rendering, images, and prefetch.
   - [Runtime and Perfetto](references/android-kotlin-perfetto.md): allocations, scheduling, trace analysis, and runtime optimization.
   - [Tracing and validation](references/tracing-validation.md): measurement design, compiler reports, frame metrics, and reporting.
5. Implement the smallest change that removes a supported cause. Preserve behavior, cancellation, accessibility, state restoration, and module ownership. An analysis-only request does not authorize edits.
6. Run the target project's relevant correctness checks. For runtime claims, compare the same journey before and after on representative physical hardware, including a lower-tier device when relevant.

## Decision rules

- Do not add `remember`, `derivedStateOf`, `@Immutable`, persistent collections, dispatcher hops, or caching mechanically. Identify the work or invalidation they would avoid.
- Prefer eliminating repeated work before moving it to another thread. Background work still competes for CPU, memory, and scheduling.
- A `suspend` function is not inherently main-safe. Inspect the implementation and distinguish blocking APIs from APIs that already dispatch internally.
- Keep related visual state coherent. Several assignments may coalesce into one frame; several stores updated in sequence are not automatically an atomic transaction. Verify observation and suspension boundaries.
- Do not use Compose observation, item counts, or list size as a completion barrier between stores. Prefer explicit operation completion and a documented publication contract.
- Protect lower-tier devices. Fast hardware can hide scheduling, allocation, and decoding costs.
- Treat reduced CPU work, fewer recompositions, and compiler skippability as evidence about a mechanism, not proof of smoother rendering.

## Report

Explain the likely or measured cause, the change and why it addresses that cause, the validation performed, and remaining behavior or measurement risks. Distinguish a code-level hypothesis, a compiler result, and a measured runtime improvement. If no device or trace was available, give a concrete next measurement instead of claiming a performance win.
