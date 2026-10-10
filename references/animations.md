# Animation cost

Use for transitions, auto-scrolling content, skeletons, or animated list cells.

Ask which phase changes on each frame. Reading animated size or padding during composition can cause composition and measurement work; layer or draw updates can often avoid those phases. Use the visual behavior the design actually requires rather than replacing every animation with a fade.

- Keep rapidly changing reads close to the consumer. Prefer the lambda form of `graphicsLayer` for changing alpha, translation, or scale when appropriate.
- Layer transforms still have GPU, clipping, compositing, and overdraw costs. Large translucent surfaces can remain expensive.
- Do not animate dimensions across many visible list items without measuring remeasurement cost.
- Scope infinite animation and carousel timers to visibility and lifecycle. Retained composition or an offscreen pager page may still run effects.
- Shared animation progress can reduce independent animation work, but broad state reads can invalidate the whole feed. Read that progress in a narrow draw/layer scope.
- Keep business work out of per-frame callbacks. Cache nontrivial animation configuration only when allocation is material.
- Use `Animatable` for interruptible/gesture sequences, transitions for related values, and simple state animation for single properties as the existing UI requires.

Validate interaction interruption, reduced-motion behavior where supported, offscreen cancellation, and frame timing. Check both UI-thread work and rendering cost.

Source: [Compose animation value and performance guidance](https://developer.android.com/develop/ui/compose/animation/value-based), [Compose phases](https://developer.android.com/develop/ui/compose/performance/phases).
