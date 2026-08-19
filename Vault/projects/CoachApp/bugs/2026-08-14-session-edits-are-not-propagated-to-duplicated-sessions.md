---
id: 2026-08-14-session-edits-are-not-propagated-to-duplicated-sessions
status: open
priority: high
reported: 2026-08-14
---

# Editing a session in a programme never offers to update its duplicates

**Jake, 2026-08-14, verbatim:** *"When adding amending/updating a session within a program (this includes
changing an exercise or chaning the session name) when I click 'Save' or 'Save edits' I should be asked
whether I want to update all duplicated sessions with the same edits. Currently I am not asked and I have
to go and make these edits 1 by 1."*

A propagation system ALREADY EXISTS (`_checkSiblingPropagation`, `_applyToAllSessions`,
`window._propagateTargets`, the `propagate-modal`). It is not firing for him. Four confirmed causes, each
independently sufficient:

1. **Renaming a session is not wired to propagation at all.** `saveEditTemplate` (`js/app-workouts.js:2565`)
   never sets `window._lastExerciseChange` and never calls `_afterTemplateExerciseSave`. Matches his words
   exactly — *"this includes… changing the session name"*.
2. **Reordering exercises isn't either** — `moveTemplateExercise` (`js/app-workouts.js:1261`), same shape.
3. **Periodization clones defeat the matcher.** `generatePhasePeriodization` names week 2+ copies
   `` `${tmpl.name} — W${week}` `` (`js/app-programs.js:1547`), and `_checkSiblingPropagation` matches on
   EXACT name equality (`js/app-workouts.js:2312`). `"X"` never equals `"X — W2"`, so for any phase built
   with "Generate weeks" the prompt is **structurally impossible**, for any edit, ever. The read-only
   Workouts page already strips that suffix cosmetically (`js/app-workouts.js:641`), so the failure is
   invisible from the UI.
4. **Editing from Library drops the programme context** the prompt requires — `openTemplate('${t.id}')`
   with `ctx = {}` (`js/app-workouts.js:788`) vs the `ctx.programId` gate at `:2302`.

Also: `_propagateTargets` maps one id per SLOT (`js/app-programs.js:2318`), so "3 other sessions" can mean
3 slots sharing 1 template, and `_propagateExerciseChangeToTemplates` writes the same target 3×.

**Root cause shared with [[2026-07-22-rows-now-carry-used-in-phase-wk-n-mon-vs-not-used-yet-plus-t]]:**
the app has no concept of "these template rows are the same session". It infers it from a name string in
one place and ignores names entirely in the other.

**Decision (Jake, 2026-08-14):** fix with a real identity column (`family_id`), not smarter name-matching.
Scope of "update all" is **the same programme only** — never a client's live assigned plan, which keeps
its own existing prompt.

**Planned session 2 of 3.** Ships together with the picker fix; both depend on the same migration.

**PARTIAL — cause 1 of 4 is closed, 2026-08-18.** `saveEditTemplate` now sets
`window._lastExerciseChange = { op: 'rename', … }` and calls `_afterTemplateExerciseSave`
(`js/app-workouts.js:2791`), so renaming IS wired to propagation. Verified in shipped code during /save.

**Cause 2 is NOT fixed** — `moveTemplateExercise` still never sets `_lastExerciseChange` nor calls
`_afterTemplateExerciseSave` (grep returns 0 matches in its body). Causes 3 and 4 not re-checked.

Deliberately left `open` rather than restatused: the closure condition below is explicit and none of it has
happened. Recorded here so the next session does not re-derive cause 1 from scratch — that near-miss is why
this note exists.

**CAUSE 2 CLOSED 2026-08-19 (`7f66634`).** Reordering now captures a `reorder` change and hands off to
`_afterTemplateExerciseSave`, so the prompt fires. The hard part was not the wiring but the permutation: a
sibling copy legitimately holds a DIFFERENT SET of exercises, so copying order_index values across would
collide with, or displace, exercises the target has and the source does not.
`_propagateReorderToTemplates` permutes only the SHARED names, into the source's relative order, reusing the
slots they already occupy — so a collision is impossible by construction. 7 red-before tests.

**Still open: causes 3 and 4.** Periodization clones defeating the name matcher, and editing from Library
dropping programme context. Neither re-checked. Row stays `open`.

**Closes when:** Jake renames a session in a periodized phase and is offered — and takes — the propagation
prompt, plus a red→green test per change shape (rename, reorder, exercise edit).
