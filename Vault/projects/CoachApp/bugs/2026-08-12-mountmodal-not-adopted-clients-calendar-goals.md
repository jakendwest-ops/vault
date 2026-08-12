---
id: 2026-08-12-mountmodal-not-adopted-clients-calendar-goals
status: open
priority: high
reported: 2026-08-12
status_detail: "found by the full-codebase architecture audit; reintroduces a previously-fixed, documented race"
---

# app-clients.js and app-calendar-goals.js never use mountModal() — reintroduces the 2026-07-04 modal-stacking race, two sites match the vulnerable shape exactly

Found by the 2026-08-12 full-codebase audit (`architecture-audit-2026-08-12.md`).

## Background

`mountModal()` (js/app-core.js:274-277) exists specifically to fix a real, previously-shipped bug: two same-id overlay nodes can stack (a fast double-tap, or an `async` function that `await`s something before creating the overlay), the *visible* one is the second, but `getElementById`/`closeModal` resolve to the buried first one — so a save reads stale/empty form values. The fix's own comment states this explicitly: *"Guarding at function entry does NOT fix the await version... has to happen at mount"* and cites the 2026-07-04 incident by date.

`app-programs.js`, `app-progress.js`, `app-runner.js`, and `app-workouts.js` have all adopted `mountModal()`. **`app-clients.js` and `app-calendar-goals.js` are the only two JS files with zero `mountModal(` calls** (confirmed via repo-wide grep).

## The sites

**js/app-clients.js** — 3 modals built via raw `document.createElement` → `document.body.appendChild`, no dedup guard at all (not even the older manual `existing?.remove()` pattern):
- `showAddClientModal` (193)
- `showUpdateEmailModal` (403)
- `showEditClientModal` (475) — **this is the exact vulnerable async shape**: it `await`s a DB fetch (421) *before* building and appending the overlay, a genuine race window, not just a fast double-tap.

**js/app-calendar-goals.js** — 4 modals, same gap:
- `showAddGoalModal` (531-535)
- `showAddMilestoneModal` (750-753)
- `showAddCheckInModal` (854-857)
- `showEditGoalModal` (916-920) — **also the exact vulnerable async shape**: `await`s `db.from('goals').select(...)` (917) before creating the overlay.

(This same file's `showAddEventModal`, `showClientAddEventModal`, and `showClientDayDetail` all guard correctly with a pre-append `?.remove()` check — the gap is specific to the 7 functions listed above, not the whole file.)

## Failure scenario

A double-tap on "Edit" for a client or a goal (or two clicks close together while the `await` is in flight) leaves two overlay nodes with the same id. `closeModal`/`getElementById` resolve to the first (buried) one — the save function reads whatever was in that first, now-invisible form, discarding what the user actually typed into the visible one. This is not hypothetical: it is the exact bug class that was reproduced and fixed once already, in a different module, on 2026-07-04.

## Suggested fix direction

Route all 7 modal-creator functions through `mountModal(overlay)` instead of raw `document.body.appendChild(overlay)`, matching the pattern already adopted in the other 4 modules.

## Documentation note

STATUS.md's Continuity block currently states the canonical modal pattern is plain `document.createElement` → `document.body.appendChild` ("Never embed in `el.innerHTML`... see `showAssignProgramModal` as canonical"), which predates `mountModal()` and doesn't mention it at all — even the cited "canonical" example doesn't call `mountModal()`. Worth updating that section to point at `mountModal()` as the current standard, since the stale doc is the plausible reason these two files never adopted it.
