---
id: 2026-08-12-app-programs-phase-writes-no-ownership-anchor
status: open
priority: high
reported: 2026-08-12
status_detail: "found by the full-codebase architecture audit; systemic gap, ~20+ call sites, single largest finding in the file"
---

# app-programs.js has no ownership-anchor helper for program_phases/program_phase_workouts — ~20+ write call sites are .eq('id', X)-only

Found by the 2026-08-12 full-codebase audit (`architecture-audit-2026-08-12.md`). This file is the heaviest `db.from()` user in the whole repo (87 calls, of which only 7 route through `dbq()`) and has no equivalent of `_verifyTemplateOwnership` (js/app-workouts.js:2560) for its own tables.

## Representative sites (not exhaustive — same shape repeats throughout the file)

- `saveEditStartDate` (623) — `client_programs` update, `.eq('id', assignmentId)` only.
- `saveProgram` update (1031) — `programs` update with no `coach_id` anchor AND no rowcount check — notably, this file's own `moveProgramToPersonal` (1120-1122) explicitly documents *why* both are required ("PostgREST returns error:null for an UPDATE that matches ZERO rows... a write that changed nothing reports a green success"), and `saveProgram` doesn't follow its own sibling's documented rule.
- `savePhase` update (1324) — `.eq('id', phaseId)` only.
- `deletePhase` (1361) — `program_phases` delete, zero anchor, despite a 6-line comment (1347-1352) in the same function about the exact cascade risk of this delete.
- `generatePhasePeriodization` (1514) — fetches the phase via `.eq('id', phaseId).single()` with no ownership check before fanning out template/exercise/phase-workout writes across every assigned client.
- `savePeriodizationConfig` (1502, 1506), `duplicatePhaseWeek` (1943, 1955), `deletePhaseWeek` (2050, 2064, 2070, 2076), `_quickAssignPhaseWorkout` (2271), `removePhaseWorkout` (2287) — same unanchored shape.
- `_deleteOwnedUnreferencedTemplates`'s own ownership-*select* (1698) has no `coach_id` filter — it trusts the caller's `programId`/`phaseIds` to already be scoped, and (separately) its "still referenced" check only looks at `program_phase_workouts`, unlike its sibling `_removeAssignmentAndClones` (267-274) which also checks `workout_logs` before deleting a client-owned clone — a plausible path to silently orphaning a logged session's `template_id` link.

## Also found in this file, same root problem

- `saveAssignProgram` (309) and `saveAssignProgramToClient` (704) insert `client_programs` from a caller-supplied `clientId` never checked against `currentUser`'s own client list.
- `_cloneTemplateForClient` (334-341) inserts `workout_templates` with `client_id: clientId` unverified (contingent on the above).

## Why this matters

No RLS SQL is tracked in this repo, so every one of these is "relying on unverified RLS as sole backstop" — this file accounts for roughly 30% of every raw, unwrapped `db.from()` call in the entire codebase, and has zero of the ownership-verification pattern its sibling module (app-workouts.js) established for the identical problem (template ownership).

## Suggested fix direction

A `_verifyProgramOwnership(programId, coachId)` / `_verifyPhaseOwnership(phaseId, coachId)` helper pair, mirroring `_verifyTemplateOwnership`, applied to every write listed above. Given the sheer number of call sites, this is a genuine refactor-scale fix, not a one-line patch — worth scoping as its own session rather than folded into unrelated feature work.
