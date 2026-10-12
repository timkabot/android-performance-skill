# Background work, battery, and thermals

Use for idle drain, repeated jobs, long wake locks, sensor/radio activity, or performance that worsens during sustained use.

## Find the owner of the work

Define the user-visible work that must complete and its latency/freshness requirements. Inspect jobs, foreground services, SDK work, receivers, alarms, sensors/location subscriptions, and persistent connections. Compare foreground, screen-off/background, and long-session behavior separately. A foreground service is not a general performance escape hatch.

| Pattern | Confirm | Fix candidate and constraint |
| --- | --- | --- |
| Same sync enqueued on every entry | Work names, queue size, run/attempt count | Unique work with a policy matching desired replacement/continuation semantics |
| Frequent tiny uploads or polls | Network timeline and bytes per wakeup | Batch deferrable work; push or longer intervals where freshness allows |
| Worker keeps retrying indefinitely | Error classification, attempts, cancellation | Bounded retry/backoff and durable progress; do not retry permanent failures blindly |
| Wake lock survives errors | Acquisition/release paths and held-time evidence | Prefer APIs that manage wakefulness; release promptly in every path with a bounded fallback |
| Sensors/location stay active offscreen | Registered listeners, frequency, lifecycle owner | Unsubscribe or reduce work only when the product's background use does not require it |
| App slows after minutes | Thermal state, CPU/GPU scheduling and sustained workload | Reduce optional work/concurrency to a sustainable level; retain essential functionality |

Use constraints for deferrable persistent work when charging/network requirements are appropriate. A periodic schedule is not an exact clock; replacing it with exact alarms merely to make tests deterministic changes power and platform behavior. Cancellation must reach loops and underlying operations, not just a surrounding coroutine.

For owned wake locks, a timeout limits damage but is not normal cleanup. Pair acquisition with release safely under exceptions and concurrent completion. Check whether WorkManager or another API already owns the lock before adding another. Recheck current Play vitals definitions and exemptions instead of hardcoding store thresholds into a code fix.

## Validate

Measure a fixed workload and a fixed idle window, with comparable radio, display, battery, and thermal conditions. Use available power rails/system tools and field metrics; short battery-percentage changes and CPU time alone are weak energy evidence. Verify completed work, freshness, and notification behavior alongside resource savings. Inspect extended use as well as cool-device bursts. Thermal APIs have device/version limitations; avoid tight polling and handle unavailable readings.

Sources: [unique work and cancellation](https://developer.android.com/develop/background-work/background-tasks/persistent/how-to/manage-work), [network batching](https://developer.android.com/develop/connectivity/minimize-effect-regular-updates), [wake-lock ownership](https://developer.android.com/develop/background-work/background-tasks/awake/wakelock/best-practices), [current wake-lock metric](https://developer.android.com/google/play/vitals/excessive-wakelock), [thermal API](https://developer.android.com/games/optimize/adpf/thermal).
