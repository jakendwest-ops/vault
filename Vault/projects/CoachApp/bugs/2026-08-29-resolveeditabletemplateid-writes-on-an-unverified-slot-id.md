---
id: 2026-08-29-resolveeditabletemplateid-writes-on-an-unverified-slot-id
status: open
priority: high
reported: 2026-08-29
status_detail: "The ownership gate checks tmpl.coach_id (the TEMPLATE); the UPDATE 14 lines later targets .eq(id, ctx.phaseWorkoutId), an id no caller verified. Verbatim the decorative-guard shape app-core.js:1084-1088 warns about. The named sibling removePhaseWorkout got this exact fix on 2026-08-22. Found by the 2026-08-29 full-file review, verified by reading :2909-2923."
---

# `_resolveEditableTemplateId` verifies the template, then writes on an unverified slot id

**js/app-workouts.js:2922-2923** — found by the weekly full-file review (Agent A), verified by reading the lines.

    const { data: repointed, error: repointErr } = await db.from('program_phase_workouts')
      .update({ template_id: cloned.id }).eq('id', ctx.phaseWorkoutId).select('id')

The ownership gate 14 lines above (`:2909-2912`) checks `tmpl.coach_id !== _resolveTemplateOwnerCoachId()`
— that protects the **template**. The write targets `ctx.phaseWorkoutId`, which neither this function nor
any of its callers has checked.

This is verbatim what `app-core.js:1084-1088` warns about: *"four sites verified clientId and then wrote
.eq('id', someOtherId), which made the guard decorative."*

**The named sibling was already fixed.** `removePhaseWorkout` got exactly this mismatched-id gate on
2026-08-22 (`tests/program-ownership-anchors-2026-08-22.spec.js:78` — *"refuses a slot belonging to a
different phase of my own"*). This site was missed — [[feedback_fix_the_class_not_the_instance]] again,
on the very class that rule exists for.

**Honest severity — "unproven at both layers", not "known-open".** The rowcount check at `:2924` means an
RLS refusal IS detected, the clone is reaped, and the user is told. But `program_phase_workouts` has **no
cross-tenant RLS probe** in `tests/`: grepping finds only fixture inserts and deletes, never a refusal
assertion. So neither layer is proven, which is the reason to close it rather than reason it away.

**Fix:** anchor the update to the template just verified —
`.eq('id', ctx.phaseWorkoutId).eq('template_id', templateId)` — so the slot must currently point at the
verified template. Cheaper and tighter than re-resolving phase to program to coach_id.

**Closes when:** the update carries the second anchor, AND a cross-tenant probe spec exists for
`program_phase_workouts` that goes RED without the anchor and GREEN with it.
