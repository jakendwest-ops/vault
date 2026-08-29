---
id: 2026-08-29-edit-sheet-save-does-nothing-for-every-cardio-set
status: open
priority: high
reported: 2026-08-29
status_detail: "The Edit affordance on a logged set renders only in the !isTable branch, which since the 2026-08-11 routing change means cardio only. The edit sheet has only Weight and Reps, and saveEditRunnerSet opens with a bare 'if (!reps) return'. No cardio set shape carries reps, so Save does nothing, every time, with no toast."
---

# The logged-set Edit sheet's Save button does nothing, 100% of the time

**js/app-runner.js:843** (the affordance) and **:1855-1858** (the handler) — found by the weekly
full-file review (Agent C).

`renderRunner` shows "✎ Edit" on a logged set **only inside the `!isTable` branch**, and
`isTable = _isPlainStrengthExercise(ex)`, which is literally `return ex.type !== 'cardio'`
(`app-runner.js:307`). So `!isTable` means cardio, and **Edit is reachable only for cardio/interval
exercises.**

But `editRunnerSet` renders a sheet with exactly two inputs — Weight and Reps — and
`saveEditRunnerSet` opens with:

    const reps = document.getElementById('wr-edit-reps')?.value.trim()
    if (!reps) return

**No cardio set shape carries `reps`.** `logRunnerSet` pushes `{distanceM}` or `{duration, distanceM}`
(`:973`, `:986`); `startIntervalTimer` pushes `{duration, distanceM}` (`:1360`); `_logIntervalPhase`
pushes `{phase, duration, distanceM}` (`:1336`). So `value="${s.reps||''}"` is always empty and Save
always hits the bare `return` — no toast, no message, the sheet just stays open.

**Sequence:** start a workout with a Skierg or interval exercise, complete a round, tap ✎ Edit on the
logged round, tap **Save** — nothing happens, ever. Delete works. Cancel works.

**Verified:** I confirmed `_isPlainStrengthExercise` is `ex.type !== 'cardio'`, that Edit sits in the
`!isTable` region, and that the handler's guard is a bare `return`. The exact branch nesting at `:827-843`
is taken from Agent C's reading.

**Corollary:** the branches at `:836-:841` (`s.distance_m`, `s.duration`, `s.leftReps`, plain weight/reps)
are dead — every non-cardio type now routes to the table.

**Fix:** either make the sheet metric-aware (duration/distance for cardio, matching `_blankTableRow`'s
dispatch), or drop the Edit affordance from the cardio list and use Delete + re-log. **Do not leave a bare
`return` on a required-field path** — this file's own convention (`toggleTableSet:430`,
`logRunnerSet:968`) is to toast.

**Closes when:** either editing a logged cardio round changes the stored value (spec: log a round, edit
it, assert the new value persists), or the affordance is removed and a spec asserts it is absent for
cardio. A bare unreachable-Save must not survive either way.
