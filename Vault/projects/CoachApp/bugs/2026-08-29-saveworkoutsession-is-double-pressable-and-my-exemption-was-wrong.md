---
id: 2026-08-29-saveworkoutsession-is-double-pressable-and-my-exemption-was-wrong
status: open
priority: high
reported: 2026-08-29
status_detail: "The reentry spec exempts saveWorkoutSession as 'manual-log twin of saveRunnerSession; navigates away on success'. BOTH halves are false: the twin disables its button synchronously at :2336, this one has no disable anywhere in its body, and closeModal is at :3020 after eight awaits. Two taps produce two workout_logs plus their exercises and sets."
---

# `saveWorkoutSession` is double-pressable, and the exemption I wrote for it was factually wrong

**js/app-runner.js:2915** — found by the weekly full-file review (Agent B), verified line by line.

`tests/reentry-guard-2026-08-28.spec.js:69` exempts it with:

    saveWorkoutSession: 'manual-log twin of saveRunnerSession; navigates away on success'

**Both halves fail on inspection.**

1. **"twin of saveRunnerSession"** — `saveRunnerSession` disables its button **synchronously at :2336**,
   before the first await:

       const saveBtn = document.querySelector('#workout-runner button[onclick="saveRunnerSession()"]')
       if (saveBtn) { saveBtn.disabled = true; saveBtn.textContent = 'Saving...' }

   `saveWorkoutSession` has **no disable anywhere in its body** — verified by grepping the whole function
   range for `disabled`/`data-busy`, which returns nothing. Its button (`:2866`) carries no
   `data-busy-text` either. They are not twins in the only respect that mattered.

2. **"navigates away on success"** — `closeModal` is at **:3020**, after **eight awaits** including three
   inserts (`:2927` client lookup, `:2936` `workout_logs`, `:2952`, `:2962`
   `workout_log_exercises`, `:2965`, `:3005` `workout_log_sets`, `:3010`, `:3011`).

**Sequence:** two rapid taps both clear the synchronous validation at `:2917-2922` and both run the full
insert chain, producing **two duplicate `workout_logs` rows plus their exercises and sets.** Coach-only
path (reachable from `renderClientWorkouts`'s "Log past session", `app-workouts.js:3092`), so no solo
exposure.

## Why this matters beyond the one function

This is the **second** documented case of me writing a confident guard justification that does not survive
reading the code — the first was the decorative `showAddExerciseToTemplateModal` registration found by the
pre-push review on 2026-08-28. Both were found by review, not by me.

**A written exemption is a claim, and this project's rule is that a claim needs a check.** The exemption
list in that spec asserts things about code ("navigates away on success", "the button goes with it") that
nothing verifies. See [[feedback_written_rules_dont_reduce_errors]] and
[[feedback_reports_success_doing_nothing]].

**Fix:** guard it, and correct the exemption text. Then consider whether the exemption list can be made
checkable rather than prose — e.g. asserting the named mechanism actually exists (a `disabled` assignment
before the first await, a `closeModal` before the first await) instead of describing it.

**Closes when:** `saveWorkoutSession` is guarded and a spec double-invokes it asserting exactly one
`workout_logs` row (RED with the guard removed, GREEN with it), AND every remaining entry in
`FROZEN_UNGUARDED` has had its stated reason checked against the code rather than trusted.
