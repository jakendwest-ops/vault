---
id: 2026-07-23-renderprogress-renderperformance-now-destroy-both-chart-arra
status: fixed-awaiting-jake
priority: low
reported: 2026-07-23
status_detail: "fixed — awaiting Jake"
---

# renderProgress/renderPerformance now destroy both chart arrays before tearing down their container

✅ **FIXED + LIVE (LOCAL) 2026-08-01 — `renderProgress`/`renderPerformance` now destroy both chart arrays before tearing down their container.** The `window.__perfCharts` reassignment cited in the original report no longer exists anywhere in the code (likely retired in an intervening refactor) — the real, still-live leak was exactly as described for the other two arrays: `_perfExerciseCharts`/`_perfSessionCharts` only ever destroyed themselves on a same-view re-render (a keystroke, a range change), never when the OUTER function replaced `#main-content`/`#progress-tab-content` via `innerHTML` on a top-level Progress tab switch or a Performance sub-tab switch. Fixed by adding a `.forEach(c => c.destroy())` + reset for both arrays at the top of both `renderProgress` and `renderPerformance`. Red→green test (`tests/ledger-fixes-2026-08-01.spec.js`, pushes a real Chart.js instance onto each array via a synthetic canvas rather than depending on the full metrics pipeline producing chartable data). — (orig) **Chart.js instances leak on tab switches.**
