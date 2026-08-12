---
id: 2026-08-12-app-workouts-ownership-anchor-inconsistency
status: open
priority: medium
reported: 2026-08-12
status_detail: "found by the full-codebase architecture audit; app-workouts.js's own comment names the intended convention explicitly, these are the exceptions to it"
---

# saveExerciseToTemplate and moveTemplateExercise skip _verifyTemplateOwnership, unlike their siblings in the same write family

Found by the 2026-08-12 full-codebase audit (`architecture-audit-2026-08-12.md`).

## The gap

`js/app-workouts.js:2553-2559`'s own comment states the intended convention by name: *"this matches the 'every sibling in the template family anchors AND verifies' convention (saveEditTemplate/deleteTemplate/saveEditExercise/deleteExercise/toggleExerciseArchived) rather than leaving this one write path as the odd one out."*

Two functions are the odd ones out:
- **`saveExerciseToTemplate`** (1916-1971, the add-exercise save) — inserts into `workout_template_exercises` after only `_resolveEditableTemplateId(templateId)` (1933), which does NOT verify ownership, only handles the fork-on-shared-slot case. No `_verifyTemplateOwnership` call, no `coachId` even resolved.
- **`moveTemplateExercise`** (1261-1286, drag-reorder) — resolves via `_resolveEditableTemplateId` only, then two raw `.update()` calls on `order_index`, no ownership check at all.

Compare their correctly-anchored siblings: `saveEditTemplateExercise` (1988-2005) and `deleteTemplateExercise` (2030-2036) both call `_verifyTemplateOwnership(targetId, coachId)` and refuse to proceed on failure.

## Mitigating context (checked, not assumed)

`tests/template-exercise-write-rls-2026-08-10.spec.js` is a live two-account behavioural probe against `workout_template_exercises` INSERT/UPDATE/DELETE, and asserts RLS refuses both an unrelated coach and a client. It doesn't cover `saveExerciseToTemplate`/`moveTemplateExercise` specifically, but it covers the same table/policy family — so this is very likely RLS-backstopped in practice, same reasoning as the already-`confirmed` sibling bug (`bugs/2026-08-01-workout-template-exercises-writes-carry-no-app-level-ownersh.md`). This is why it's filed as Medium, not High — a real inconsistency against the file's own documented rule, but probably not a live hole.

## Suggested fix direction

Add the same `_verifyTemplateOwnership` call both functions' siblings already have — closes the inconsistency and removes the app-level reliance on RLS-only for these two specific writes.
