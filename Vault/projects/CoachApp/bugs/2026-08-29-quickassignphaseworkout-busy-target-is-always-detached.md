---
id: 2026-08-29-quickassignphaseworkout-busy-target-is-always-detached
status: open
priority: low
reported: 2026-08-29
status_detail: "_quickAssignPhaseWorkout is the one guardReentry member not invoked from an inline onclick. Its caller _pickWorkout calls _closeWorkoutPicker() synchronously first, so window.event.currentTarget resolves to an already-detached picker row div. _setBusy sets aria-busy on an off-screen node and 'disabled' in el is false for a div, so the guard gives zero feedback and zero disabling. Cannot strand anything - a feedback gap, not corruption."
---

# `guardReentry`'s busy target is always a detached node for `_quickAssignPhaseWorkout`

**js/app-core.js:87-91** with **js/app-programs.js:2519-2522** — found by the weekly full-file
review (Agent C).

`_quickAssignPhaseWorkout` is the **one member of the eight that is not invoked from an inline
`onclick`**. Its caller:

    function _pickWorkout(templateId) {
      const s = _workoutPickerState
      if (!s) return
      _closeWorkoutPicker()                  // removes the picker modal, synchronously
      _quickAssignPhaseWorkout(s, templateId)
    }

`_pickWorkout` is the inline handler on the picker **row `<div>`** (`app-programs.js:2488`). Inside the
wrapper, `_busyTarget()` reads `window.event.currentTarget` -> that row div -> `closest('button')` finds
no button ancestor -> returns the div. But `_closeWorkoutPicker()` has **already detached it**.

Net: `aria-busy` is set on an off-screen node, and `'disabled' in el` is false for an `HTMLDivElement`,
so nothing is disabled either. **Zero visual feedback and zero disabling** while two ownership checks, a
slot re-check and an insert run. It cannot *strand* anything (a div has no `disabled`), so this is a
feedback gap, not corruption — but it is where the `window.event` assumption breaks, which is what to
know before the next non-onclick member is added.

**Fix:** either have `_pickWorkout` pass its own element through, or — simpler and more correct —
**make `_pickWorkout` the guarded name instead.** It is the gesture, and it is what the user actually
presses.

**Also, a smaller documentation error found alongside:** the justification comment at
`app-core.js:96-98` cites `moveTemplateExercise`'s up/down arrows as the reason `_setBusy` remembers the
prior disabled state. **`moveTemplateExercise` is not a `guardReentry` member.** The logic is right; the
example is not, and the next reader will go looking for a member that does not exist.

**Closes when:** the busy target resolves to a live element for every one of the eight registrations,
asserted by a spec that checks `aria-busy` actually appears on something in the document — not merely
that the guard blocked. And the comment names a real member.
