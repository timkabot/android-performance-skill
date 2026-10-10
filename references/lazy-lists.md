# Lazy lists, grids, and nested layouts

Use for feed scrolling, expensive cells, pagination, key collisions, or restoration problems.

## Identity and reuse

- Use stable unique keys when items can move or change. Prefer a durable item or placement identifier.
- For mixed feeds, provide `contentType` by structural type, not by unique item ID.
- Random IDs or timestamps created on every mapping pass avoid collisions by destroying identity. They can discard remembered state and scroll anchors.
- When an entity appears in several sections, combine section identity and item identity. If the backend supplies no durable placement ID, a deterministic occurrence suffix may resolve collisions, but can change when duplicates reorder. Document that limitation.
- Test key uniqueness and identity across refresh and reordering when key generation changes. Stability across identical input alone does not establish stability across edits.
- Keys used with Android saved item state must be supported by `Bundle`.

Illustrative feed with application-specific row models:

```kotlin
LazyColumn {
    items(
        items = rows,
        key = { it.id },
        contentType = { it.type },
    ) { row ->
        FeedRow(row = row, onClick = { onRowClick(row.id) })
    }
}
```

## Item work

- Precompute expensive formatting, grouping, sorting, and lookup indexes where their inputs change.
- Avoid resolving dependencies, building large analytics payloads, or attaching frame listeners for each composed item.
- Size images to their display slot and keep placeholders close to final dimensions.
- Dedupe impression events by the intended visibility/session semantics. Recomposition is not an impression.
- Read only visible layout metadata in scroll collectors; do not scan the entire data set per pixel.
- Keep pagination dedupe and in-flight/last-page checks in the state owner. A UI threshold can emit repeatedly as layout changes.

## Nesting

Flatten unbounded same-axis scrolling into a single lazy container where practical. Bounded nested scrolling can be valid; inspect measurement constraints and UX before rewriting it. Cross-axis carousels still need sensible dimensions, stable outer/inner keys, and measured item cost. For a handful of fixed items, an ordinary `Row` or `Column` may be simpler.

## Restoration

Inspect the navigation and saveable-state lifecycle first. Standard remembered lazy state can handle restoration when the state holder survives or is restored correctly. Initial position arguments are initializers, not a command to restore every time a screen returns.

When data loads asynchronously, coordinate any explicit restoration with data readiness and perform it once per restoration request. Avoid saving temporary layout resets over the user's position, and avoid repeatedly forcing the list back while the user scrolls. Validate back navigation, process recreation where supported, refresh, and reordered or removed items. Reuse an existing project helper if it provides these guarantees; no named helper is required.

Source: [Compose lazy lists and grids](https://developer.android.com/develop/ui/compose/lists).
