---
id: 2026-08-22-no-delete-in-the-programs-family-is-rowcount-checked
status: open
priority: medium
reported: 2026-08-22
status_detail: "found by multi-agent-review Agent A while reviewing the app-programs ownership-anchor work; deliberately NOT folded into that commit — it is its own class"
---

# Every DELETE in app-programs reports success when it deletes nothing

PostgREST returns `error: null` for a DELETE that matches **zero** rows — the same property this repo
already documents for UPDATE at `moveProgramToPersonal`, and the reason `.select()` + a rowcount check
is the house rule.

The 2026-08-22 ownership-anchor commit applied that rule to three UPDATEs (`saveEditStartDate`,
`saveProgram`, `savePhase`). **It applied it to none of the DELETEs those same gates guard:**

- `removePhaseWorkout` — `dbq(... .delete().eq('id', pwId))`, `if (error) return` only, then
  re-renders as though it succeeded
- `deletePhase` — the `program_phase_workouts` delete and the `program_phases` delete
- `deleteProgram` — `program_phase_workouts.delete().in('phase_id', phaseIds)`
- `_cleanupPhaseWeeksBeyond`
- `deletePhaseWeek`

A refused DELETE at any of these tells the user it worked. This is the "reports success while doing
nothing" class, which is the dominant OS bug class here.

## Why it was not fixed in the same commit

Two reasons, both deliberate. It is **pre-existing**, not introduced by the ownership work — so folding
it in would have mixed a security change with a behavioural one in the same diff. And it is a genuine
class of its own (5+ sites across 5 functions, each with a different "what should the user see when the
delete was refused?" answer). The ownership commit was already at 45 writes across 23 functions.

## Fix direction

`.select('id')` on every delete, then branch on rowcount. Note the correct branch differs per site:
`deleteProgram`'s zero-row delete of phase slots is LEGITIMATE when a program has no phases, so a bare
`length !== 1` check would be wrong there. Do not apply one template mechanically — that is how a
guard ends up refusing the legitimate user.

**Closes when:** every DELETE in js/app-programs.js either checks its rowcount or carries a comment
saying why zero rows is expected, and a test proves at least one refused delete no longer reports
success.
