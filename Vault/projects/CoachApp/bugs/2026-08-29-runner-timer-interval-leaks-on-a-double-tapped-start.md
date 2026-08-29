---
id: 2026-08-29-runner-timer-interval-leaks-on-a-double-tapped-start
status: open
priority: high
reported: 2026-08-29
status_detail: "_startRunnerTimerTick starts a 1Hz setInterval with no clearTimer first, while its adjacent sibling _startRunnerDraftSafetyNet clears at :138. Double-tapping Start orphans the first interval for the life of the page - it survives both save and discard. Also: launchRunner is a second escape from the reentry spec's INSERT-based enumeration."
---

# A double-tapped ▶ Start orphans a 1 Hz interval for the life of the page

**js/app-runner.js:58-67** — found by the weekly full-file review (Agent C), verified by reading both
functions side by side.

    function _startRunnerTimerTick() {
      _runner._timerInterval = setInterval(() => { ... }, 1000)     // no clear first
    }

Its sibling **one function below**, called on the very next line at both call sites (`:54-55`, `:183-184`),
does clear first:

    function _startRunnerDraftSafetyNet() {
      _runnerDraftSafetyNetInterval = clearTimer(_runnerDraftSafetyNetInterval)   // :138

Straight copy-paste drift between two adjacent, back-to-back-invoked functions.

**Sequence:** Workouts to "▶ Start workout" to the `runner-setup` modal, then **double-tap ▶ Start**
(`app-workouts.js:3188`). `_startFreshRunner`'s first suspension is the `client_1rms` fetch at `:27`, and
it does not remove `#runner-setup` until `:45` — so the button stays live and enabled across a full
network round-trip. Both calls run. Line `:47` assigns a **new** `_runner` object, so the first call's
`_timerInterval` id becomes unreachable. `discardRunner:2289` and `showRunnerFinish:2093` clear only
`_runner._timerInterval` — the second one. The first ticks for the life of the page, surviving both save
and discard.

**This is a second escape from the re-entry spec's scanner.** `launchRunner` is not a `guardReentry`
member and is not on `FROZEN_UNGUARDED`, because it inserts nothing and the spec enumerates by INSERT.
The first escape was `saveWorkoutSession`. **The enumeration criterion needs revisiting: "a double press
causes damage" is broader than "a double press inserts a row."**

**Fix:** `_runner._timerInterval = clearTimer(_runner._timerInterval)` before the `setInterval`, matching
`:138`. Clearing the OLD object's id additionally requires capturing it before `:47`, or guarding
`launchRunner` against re-entry.

**Closes when:** a spec double-invokes the start path and asserts exactly one live interval remains —
RED before the clear, GREEN after.
