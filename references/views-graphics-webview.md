# Views, graphics, interoperability, and WebView

Use when the app is not exclusively Compose, main-thread composition looks cheap, or web/media surfaces are slow.

## Rendering diagnosis

| Pattern | Evidence | Fix candidate |
| --- | --- | --- |
| Long measure/layout without many recompositions | Layout Inspector plus matching trace | Reduce repeated measurement, unstable dimensions, expensive text/layout work |
| Main is fast but frame misses persist | RenderThread/GPU/compositor and scheduling tracks | Reduce proven overdraw, large offscreen layers, blur, texture upload, or rendering load |
| RecyclerView rebinds nearly everything | Bind/inflate counts, adapter notifications, diff work | Correct identity/content comparison and targeted updates; move expensive diff work appropriately |
| Mixed Views/Compose loses state or rebuilds cells | View/composition creation and disposal lifecycle | Choose pooling/lifecycle strategy for that host; avoid obsolete manual disposal per recycled bind |
| AndroidView repeatedly recreated in a lazy list | Factory/update/reset/release calls | Use supported reuse overload where appropriate, resetting all item-specific state and resources |
| Text-heavy rows dominate CPU | Text shaping/layout traces, locale/font scale, repeated formatting | Cache/prepare only with correct text/style/constraint keys; preserve typography and accessibility |

Do not rewrite Views to Compose, disable hardware acceleration, or reduce refresh rate merely because frames are slow. Subcomposition, layout, rendering, image upload, and platform IPC have distinct costs. Reuse requires correct reset semantics; leaking prior WebView/player/item state is not an acceptable optimization.

## WebView

Split WebView engine startup from navigation, network, JavaScript, layout/paint, and bridge calls. Check implicit initialization by SDKs/providers and user-agent access. Use WebView-specific debugging for page work and Android traces for host work; a blank first WebView frame does not prove useful web content is ready.

For early initialization, consult the installed AndroidX WebKit version and the supported `WebViewCompat.startUpWebView` contract. Overloads and stability changed across releases. Calling WebView APIs immediately afterward can block on startup and erase the benefit. Measure any prewarming against launch contention and memory; do not move ordinary WebView operations to arbitrary threads or use undocumented reflection/user-agent tricks.

Handle renderer termination and lifecycle/resource cleanup. Preserve user state across navigation/process recreation where supported; do not share a live WebView across incompatible Activity lifetimes just to avoid construction.

## Validate

Compare creation/bind counts, frame phases, memory, and first useful content. Exercise cell reuse, navigation away/back, font scale, resizing, and renderer failure for relevant fixes. Keep Android frame timing and web content readiness as separate metrics.

Sources: [Compose in Views](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/compose-in-views), [Views in Compose](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/views-in-compose), [rendering diagnosis](https://developer.android.com/topic/performance/vitals/render), [WebView startup](https://developer.android.com/develop/ui/views/layout/webapps/optimize-webview-startup), [WebViewCompat API](https://developer.android.com/reference/androidx/webkit/WebViewCompat).
