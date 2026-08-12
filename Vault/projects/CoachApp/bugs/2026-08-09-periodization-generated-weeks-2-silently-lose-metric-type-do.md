---
id: 2026-08-09-periodization-generated-weeks-2-silently-lose-metric-type-do
status: open
priority: high
reported: 2026-08-09
---

# periodization-generated weeks 2+ silently lose metric_type, downgrading every cardio/interval/jump/timed exerc

**🔴 LIVE BUG — periodization-generated weeks 2+ silently lose `metric_type`, downgrading every cardio/interval/jump/timed exercise to plain weight-and-reps.** Found 2026-08-09 during superset scoping, then **verified directly in the source, not inferred**: `generatePhasePeriodization` (`app-programs.js:1527-1531`) builds its clone rows with `exercise_type` but **no `metric_type`**. Its sibling `_cloneTemplateForClient` (`app-programs.js:350`) carries it, with a comment describing this exact failure verbatim — *“Omitted here until 2026-07-22, so every ASSIGNED copy silently fell back to weight_reps”* — and `_cloneSharedMasterTemplate` (`app-workouts.js:2341`) carries it too. **The same fix was applied to two siblings and this third was missed** — fix-the-class-not-the-instance, now at least the 6th time in this codebase. **Real-world effect:** build a periodized phase containing a SkiErg interval or a Box Jump, generate weeks 2–4, and every generated week renders and runs those as weight×reps — no duration, no distance, no HR/watts capture, no jump height. Week 1 is correct, so it looks like the later weeks were built wrong. Silent at every layer: the source select is `workout_template_exercises(*)` so the value was always available, and a missing key is not an error in JS, in the insert, or in Postgres. **Fix is one line**, in a file the superset work will edit anyway.
