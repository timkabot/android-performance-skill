# Android performance landscape and skill coverage review

Research date: **2026-10-12**. Scope: publicly documented Android application problems, engineering case studies, Google guidance, research papers, and developer talks that can improve this skill.

This is an evidence review, not a market-share study or a benchmark ranking of competing apps. No proprietary telemetry or APK benchmarking was performed. Published success stories have selection bias; historical open-source datasets and Wear OS results cannot establish the prevalence of a problem in today's phone apps.

## Main findings

The original skill concentrated on Compose rendering and state publication. The largest coverage gaps were startup dependencies, ANR completion contracts, memory/resource ownership, data volume and persistence, background power, release artifacts, and mixed View/WebView surfaces.

The review supports broadening diagnosis before prescribing a fix. It does not support a universal ranking of libraries, a guaranteed percentage improvement, or an assumption that every code smell is a runtime bottleneck. CPU time, responsiveness, memory, download size, and energy are related but distinct outcomes.

## What production cases demonstrate

| Evidence | Reported outcome or observation | Lesson adopted | Limit |
| --- | --- | --- | --- |
| [Android Calendar, Google case study](https://developer.android.com/topic/performance/baselineprofiles/case-study-android-calendar) | Median interactive startup 775 → 644 ms; cold startup 1,058 → 901 ms; janky-frame rate improved for the studied calendar views | Profile the journeys people actually use and account for release cadence / Cloud Profiles | Specific app, views, artifacts and rollout comparison |
| [Gmail Wear OS, Google case study](https://developer.android.com/topic/performance/appstartup/case-study-gmail-wear) | Loading UI competed for startup CPU; the team reports a 50% startup improvement from its loading-state changes | Inspect competing work after launch, including supposedly cosmetic work | Constrained watch workload; not evidence that phone apps should retain splash screens longer |
| [Meta Engineering, 2025-10-01](https://engineering.fb.com/2025/10/01/android/accelerating-our-android-apps-with-baseline-profiles/) | Reports improvements across metrics/apps and occasional regressions when profile size increased | Treat profile coverage and compiled footprint as a measured tradeoff | Large-app infrastructure and workload; no transferable promised gain |
| [Google Performance Spotlight recap, 2025-11-21](https://android-developers.googleblog.com/2025/11/fully-optimized-wrapping-up-performance.html) | Reports Reddit improvements after R8 full mode and Disney+ gains after removing an optimization-disabling flag | Audit effective release configuration before widespread UI micro-optimizations | Google-published case summaries, not an independent reproduction |

## Coverage across application categories

The following is our engineering synthesis of workload shapes, not measured market prevalence. A category determines candidate journeys; traces determine the actual bottleneck.

| Workload | Representative journey | Additional risks worth checking |
| --- | --- | --- |
| Commerce, social, news | Launch → feed → detail → return → refresh | Image footprint, duplicate loads, SDK startup, list identity, expensive placeholders, retention |
| Productivity and offline apps | Open large local dataset → filter → edit → sync | SQL plan, transaction scope, migrations, invalidation, durability |
| Media, maps, communication | Enter heavy surface → navigate away → background | Native resources, renderer/player lifetime, continuous sensors, persistent network work |
| Hybrid apps | Native launch → first web screen → back | Implicit WebView initialization, web readiness, renderer termination, mixed UI ownership |
| Wear and constrained devices | Cold start → short interaction → idle | CPU contention, memory pressure, startup animation, background energy |
| Long-session apps | Repeated navigation/scroll plus background cycles | Retained resources, cache growth, thermal limits, accumulating jobs/listeners |

## Research that changed the rules

1. **[Rua and Saraiva: A large-scale empirical study on mobile performance: energy, run-time and memory](https://link.springer.com/article/10.1007/s10664-023-10391-y)** — published online 2023-12-27, journal volume 2024. The study covers 1,322 versions of 215 Android apps and more than 27,900 test scenarios. It reports context-dependent effects of practices and weak correspondence between static issue priority and measured outcomes. We adopted multidimensional guardrails and rejected automatic “smell → optimization” rules. Its corpus, tools, hardware, historical versions and short automated scenarios limit generalization to current apps and sustained usage.
2. **[DroidLeaks](https://arxiv.org/abs/1611.08079)** — 2016 preprint. Its abstract reports 298 fixed resource leaks mined from historical open-source app revisions. It motivates checking explicit resource ownership and abnormal paths in addition to heap retention. It is not a current market leakage rate. Review depth here: abstract and bibliographic metadata.
3. **[PlumbDroid: Automated Repair of Resource Leaks in Android Applications](https://arxiv.org/abs/2003.03201)** — preprint 2020, revised 2022, linked journal publication 2022. Its repair model emphasizes valid release locations across event-driven control flow. We adopted lifecycle-aware cleanup and validation that resources remain usable while needed. The study does not validate arbitrary agent-generated fixes. Review depth here: abstract and bibliographic metadata.

## Google talks, workshops, and technical articles

| Resource | Type / date | Relevant content and how it is used |
| --- | --- | --- |
| [More performance tips for Jetpack Compose — Ben Trengrove](https://www.youtube.com/watch?v=ahXLwg2JYpc) | Android Dev Summit, 2022-10-24 | Deferred state reads, stability, fully-drawn reporting. Reconcile old compiler assumptions with current strong skipping before applying |
| [Practical performance problem solving in Jetpack Compose](https://developer.android.com/codelabs/jetpack-compose-performance) | Google I/O 2024 workshop/codelab; [event catalog](https://io.google/2024/explore/?q=android) | Readable worked investigation of decoding, binder effects, subcomposition and frame measurement. Feeds the lazy-list diagnostic additions |
| [Android Studio profiler session — Mayank Jain and Yasser Dbeis](https://www.youtube.com/watch?v=ELKKWDPCLXU) | Google I/O, published 2024-05-16 | Official session description covers profiler capabilities and performance analysis. Tool-selection context rather than API authority |
| [App performance improvements — Ben Weiss and Alice Yuan](https://www.youtube.com/watch?v=xMylnxek5a4) | Official Android Developers recording, published 2025-11-20; description labels Google I/O 2025 | End-to-end measurement, Macrobenchmark/UiAutomator, R8, vitals and wake locks; supports broader routing |
| [Rules about performance tools](https://www.youtube.com/watch?v=bvXWQ2NidmQ) | Android Developers Backstage, 2024-11-26; podcast, not a conference talk | System health, performance/energy tradeoffs, and tool choice; supports evidence-led diagnosis |
| [Fully Optimized: Performance Spotlight Week](https://android-developers.googleblog.com/2025/11/fully-optimized-wrapping-up-performance.html) | Google technical roundup, 2025-11-21 | Release optimization and newer Compose prefetch/cache behavior; version-check requirements |
| [Android Performance Analyzer](https://developer.android.com/android-performance-analyzer) | Current official tool documentation, checked 2026-10-12 | Additional system/frame profiling path. Optional, with current compatibility requirements |

**Video review limit:** official indexed descriptions, speaker/event metadata, and chapter lists were available; direct video-page retrieval was throttled or failed. Full recordings/transcripts were not reviewed. Implementation guidance is grounded in the readable documentation/codelab links below, not inferred from unwatched demonstrations. Session descriptions and historical talks are discovery aids; library/platform documentation governs APIs.

## Primary diagnostic sources and resulting coverage

| Source | New or strengthened decision |
| --- | --- |
| [Startup analysis](https://developer.android.com/topic/performance/appstartup/analysis-optimization) | Trace launch and first use; distinguish readiness metrics |
| [ANR diagnosis](https://developer.android.com/topic/performance/anrs/diagnose-and-fix-anrs) | Classify deadline/owner, chase lock and binder counterpart, distrust late idle stacks |
| [SharedPreferences API](https://developer.android.com/reference/android/content/SharedPreferences) | Pending async writes can still stall lifecycle transitions |
| [Memory management](https://developer.android.com/topic/performance/memory/manage-app-memory) | Separate churn, retention, native footprint and process loss |
| [System memory troubleshooting](https://developer.android.com/topic/performance/memory/guide/system-wide-memory) | Inspect whole-process/system pressure, not just Java heap |
| [SQLite performance](https://developer.android.com/topic/performance/sqlite-performance-best-practices) | Use data scale, query plans and transaction evidence |
| [OkHttp events](https://github.com/square/okhttp/blob/master/docs/features/events.md) and [caching](https://github.com/square/okhttp/blob/master/docs/features/caching.md) | Per-call stage attribution, repeated/missing events, freshness and ownership |
| [Network troubleshooting](https://developer.android.com/develop/connectivity/troubleshoot-network-issues) | Track request frequency, bytes and source |
| [Regular network updates](https://developer.android.com/develop/connectivity/minimize-effect-regular-updates) | Batch only work that can tolerate deferral |
| [WorkManager work management](https://developer.android.com/develop/background-work/background-tasks/persistent/how-to/manage-work) | Deduplicate work according to business semantics; cancellation/attempts |
| [Wake-lock best practices](https://developer.android.com/develop/background-work/background-tasks/awake/wakelock/best-practices) and [vitals definition](https://developer.android.com/google/play/vitals/excessive-wakelock) | Inspect acquisition/release and current exemptions; do not freeze policy thresholds into the skill |
| [Thermal API](https://developer.android.com/games/optimize/adpf/thermal) | Sustained workloads and unsupported readings; avoid tight polling |
| [R8 optimization](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization) | Effective release configuration and version-specific DSL |
| [Profile measurement](https://developer.android.com/topic/performance/baselineprofiles/measure-baselineprofile) and [debugging](https://developer.android.com/topic/performance/baselineprofiles/debug-baseline-profiles) | Explicit compilation modes, installed artifact verification and experiment limits |
| [Compose in Views](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/compose-in-views) and [Views in Compose](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/views-in-compose) | Host-specific disposal, pooling, reuse/reset correctness |
| [WebView startup](https://developer.android.com/develop/ui/views/layout/webapps/optimize-webview-startup) and [WebViewCompat](https://developer.android.com/reference/androidx/webkit/WebViewCompat) | Versioned async initialization; immediate use may negate prewarming |

Sources were inspected as pages/API text or official repository documents, except the explicitly limited video and paper entries. This review preserves links and bounded conclusions rather than copying articles into the skill.

## Changes made to the skill

- Added seven focused guides: startup; ANRs/contention; memory/resources; storage/network; background/power; release optimization; Views/graphics/WebView.
- Added symptom-based routing and required evidence labels: measured, code-supported, hypothesis.
- Added concrete diagnostic tables with confirmation steps and constrained fixes.
- Extended lazy-list guidance with subcomposition, local-image decoding, repeated system listeners, and cache-window tradeoffs.
- Extended validation with guardrail metrics, cohorts, denominator checks, profiling overhead and falsifiable hypotheses.
- Kept specialist details out of the entry point. The bibliography is optional reading, not mandatory context for every invocation.

## What this does not prove

The guides improve explicit coverage; no Android application was profiled or optimized during this skill update. Static package checks and a manual scenario review cannot establish better model outcomes. The [evaluation scenarios](../evals/performance-scenarios.md) define a reproducible next behavioral comparison against the previous skill revision. Native engine rendering, codec tuning, ML inference, and device-driver failures still require domain-specific investigation beyond this general app skill.
