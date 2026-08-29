---
id: 2026-08-29-soloclientid-interpolated-raw-into-onclick-at-2-sites
status: open
priority: medium
reported: 2026-08-29
status_detail: "window._soloClientId is initialised to null (app-core.js:167) and only set if a maybeSingle lookup succeeds, so null is a real state. Two sites interpolate it raw into an onclick string, producing startWorkoutRunner('null', id). app-dashboard.js:637 already carries the correct guard, so the fix pattern exists."
---

# `window._soloClientId` is interpolated raw into an onclick at 2 sites

Found by the weekly full-file review (Agent B), **which reported one site and called it the only
consumer outside app-core. It is not** — I grepped the class and found a second unguarded site plus a
correctly-guarded model.

`window._soloClientId` is initialised to `null` (`app-core.js:167`) and only assigned
`if (soloRec)` from a `maybeSingle()` whose error is logged but not fatal (`app-core.js:780-782`). **So
null is a reachable state**, not a theoretical one.

| Site | State |
|---|---|
| `app-dashboard.js:637` | **guarded** — `if (!clientId) { ...'Personal account not set up yet.'; return }` |
| `app-workouts.js:1234` | **unguarded** — `onclick="startWorkoutRunner('${window._soloClientId}','${id}')"` |
| `app-programs.js:813` | **unguarded** — `onclick="saveAssignProgramToClient('${programId}','${isSolo ? window._soloClientId : ''}')"` |

**Why the failure is nasty at `app-workouts.js:1234`:** the button renders as
`startWorkoutRunner('null', '<id>')`. Tapping it launches a full runner bound to the **string** `"null"`.
`_startFreshRunner` fires `.eq('client_id','null')` — an invalid uuid, discarded silently because that
call does not go through `dbq` — so **the runner looks completely normal**. The failure surfaces only at
`saveRunnerSession` (`:2345-2350`): *"Could not verify this client — please refresh and try again"*,
**after the entire session has been logged.** The worst possible place to fail.

**Fix:** resolve via `await _getCurrentClientId()` (`openTemplate` is already async) and omit the button
when it is null, matching the guard `app-workouts.js:625-626` already uses. Same shape at
`app-programs.js:813`.

**Closes when:** both sites are guarded, and a spec renders each with `window._soloClientId = null` and
asserts the affordance is absent rather than present-and-broken. The guarded sibling at
`app-dashboard.js:637` is the model — [[feedback_fix_the_class_not_the_instance]].
