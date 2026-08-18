---
id: 2026-08-17-renderclientweight-leaks-a-chart-on-every-save
status: open
priority: medium
reported: 2026-08-17
status_detail: "Weekly full-file review. Verified by reading both twins — renderProgressWeight destroys, renderClientWeight does not."
---

# The coach's client Weight tab leaks a live Chart on every save

`renderClientWeight` (`js/app-progress.js:719-919`) replaces `el.innerHTML` at `:740` — detaching
`#weight-chart` — then calls `_renderMetricChart('weight-chart', ...)` at `:896`, **without ever calling
`_destroyManagedCharts()`**.

Both of that helper's guards miss on a rebuild: `Chart.getChart(el)` resolves the NEW canvas (undefined),
and `_activeCharts.filter(c => c.canvas !== el)` compares against the new element, so the old entry
survives. This is the mechanism spelled out in `renderProgressPBs`' own comment (`:2443-2448`), written
when the same leak was found there.

Its twin `renderProgressWeight` does destroy (`:1432`), as does every other chart entry point: `:601`,
`:1116`, `:1179`, `:1324`, `:1898`, `:2325`, `:2449`. This one was missed.

## Repro

Coach → a client → **Weight** tab → "Save entry" five times. `saveWeightLog:984` re-enters
`renderClientWeight` each time; `_activeCharts.length` grows by one per save, each a live Chart bound to
a detached canvas with its listeners and animation loop running.

Also via `deleteWeightLog:1023`, `saveWeightGoals:1005`, and Weight↔Overview round trips
(`app-clients.js:313` does `content.innerHTML = ''` with no destroy). Bounded only by eventually visiting
a tab that does destroy. One line.
