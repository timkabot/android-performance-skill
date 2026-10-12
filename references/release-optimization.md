# Release configuration, profiles, and footprint

Use for broad audits, first-install/update regressions, large artifacts, or behavior that differs between debug and release.

## Inspect the artifact users receive

Record AGP, Kotlin/Compose compiler, app and SDK versions, release flags, ABI, and packaging mode. Inspect merged configuration and the produced APK/APKs, not just one Gradle file. Do not prescribe a current DSL or compiler flag without checking the installed version.

| Pattern | Confirm | Fix candidate / validation |
| --- | --- | --- |
| R8 is nominally enabled but optimization suppressed | Effective rules, `-dontoptimize`, broad keeps, compatibility flags | Restore intended optimization and narrow rules; test reflection, JNI, serialization and entry points in release |
| First-run JIT/class loading dominates | Startup/interaction trace and profile coverage | Generate representative Baseline Profiles; verify packaging/installation and benchmark compilation mode |
| Profile covers splash but not useful content | Collection journey and readiness boundary | Extend to important interactions; keep Startup Profile focused on startup |
| Bigger profile regresses memory/startup | Compiled footprint and before/after latency | Select hot paths using evidence; more methods is not automatically better |
| Download/install footprint grows | Delivered splits, DEX, assets/resources and native libraries | Remove unused dependencies/resources, right-size assets, inspect ABI delivery without dropping supported devices |
| New SDK causes multiple regressions | Dependency/config diff, manifest providers, threads, jobs, resources and keeps | Attribute and configure/replace only the offending work; validate SDK behavior |

Baseline Profiles guide ahead-of-time compilation of important code paths; Startup Profiles guide DEX layout for startup. They have different purposes. Use representative user journeys beyond launch where appropriate. Verify profile inclusion and application rather than assuming generation equals deployment.

Do not blindly copy another app's percentage improvement. Library and Cloud Profiles, release cadence, device mix, and baseline compilation change the result. Profile/config changes are baked into artifacts: a runtime feature flag cannot switch the compiled optimization on and off within an identical APK. Compare controlled artifacts locally and use a suitable release comparison for field impact.

## Validate

Run release correctness checks covering dynamic/reflected code; retain mappings/symbols for actionable reports. Compare startup, critical interactions, footprint, and memory under explicit compilation modes. Separate compressed download size, installed size, and runtime memory. A smaller artifact is not sufficient evidence of faster frames; a faster debug build is not release evidence.

Sources: [R8 optimization](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization), [profile measurement](https://developer.android.com/topic/performance/baselineprofiles/measure-baselineprofile), [profile debugging and experiment limits](https://developer.android.com/topic/performance/baselineprofiles/debug-baseline-profiles), [Meta's profile tradeoffs](https://engineering.fb.com/2025/10/01/android/accelerating-our-android-apps-with-baseline-profiles/), [Google's optimization case roundup](https://android-developers.googleblog.com/2025/11/fully-optimized-wrapping-up-performance.html).
