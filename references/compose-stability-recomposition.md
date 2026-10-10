# Compose stability and recomposition

Use when expensive subtrees rerun, parameter identity changes, or models carry incorrect stability contracts.

## Establish the compiler baseline

Inspect the Kotlin/Compose compiler configuration and report destinations. Do not assume a particular plugin, report path, or skipping setting.

With strong skipping, restartable composables can skip even with unstable parameters: unstable arguments use identity comparison and stable arguments use equality. Captured lambdas are also memoized by the compiler. Consequently, a fresh but equal unstable model may still cause work, while manually remembering every callback may add no value. Strong skipping is enabled by default from Kotlin 2.0.20; inspect the actual module configuration.

See [strong skipping](https://developer.android.com/develop/ui/compose/performance/stability/strongskipping).

## Models

- `@Immutable` and `@Stable` are promises, not runtime enforcement. Incorrect promises can produce stale UI.
- Inspect nested values and collection ownership. A read-only `List` interface does not prove the backing collection is immutable.
- Persistent collections can help express immutable state, especially if the project already uses them. Do not add a dependency or change every model without evidence.
- Observable snapshot collections are valid Compose state. Distinguish tracked mutation from ordinary mutable collections that Compose cannot observe.
- Convert or derive models at the owning state boundary, rather than repeatedly in item composition.
- For models from modules without Compose compilation, inspect reports before adding wrappers or stability configuration. Both have maintenance costs.

## Reads and effects

- Pass the fields a child needs; avoid routing one large frequently changing state object through every item.
- Read scroll or animation state in the smallest relevant scope, or in layout/draw lambdas when the visual change permits it.
- `remember` caches a calculation for its keys. Missing keys can show stale data; remembering heavy mapping still executes it on the composition thread on cache misses.
- `derivedStateOf` is useful when a frequently changing input produces a less frequently changing result. It adds overhead for ordinary cheap derivation.
- Key effects by the operation's identity. Whole-state keys restart work too often; constant keys can retain obsolete inputs.
- Use `rememberUpdatedState` when a long-lived effect must see a current callback without restarting.
- Avoid writing state during composition after reading it. Keep mutations in event handling or appropriate effects.

## Verify

Inspect the affected classes/composables in compiler reports and use recomposition tooling to locate activity. Then inspect frame traces: skipping a cheap function is not automatically worth expensive equality checks or model copying.

Source: [stability fixes](https://developer.android.com/develop/ui/compose/performance/stability/fix), [Compose phases](https://developer.android.com/develop/ui/compose/performance/phases).
