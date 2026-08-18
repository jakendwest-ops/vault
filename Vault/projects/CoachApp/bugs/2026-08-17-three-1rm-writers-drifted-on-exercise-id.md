---
id: 2026-08-17-three-1rm-writers-drifted-on-exercise-id
status: open
priority: low
reported: 2026-08-17
status_detail: "Weekly full-file review."
---

# Two of the three 1RM writers omit `exercise_id`, so runner-logged 1RMs break on rename

`save1RM` (`js/app-progress.js:380`) writes `exercise_id: picked.id`.

`saveRunnerOneRM` (`js/app-runner.js:2628`) and `_savePostSessionOneRM` (`:2509`) both insert WITHOUT it —
even though `launchRunner:29` builds `oneRMById` keyed on `exercise_id` and `_lookupClientOneRM:1925`
prefers it over the name.

Rows written from the runner therefore survive only on the name fallback, and break silently the moment
an exercise is renamed — the precise failure `exercise_id` was added to prevent.

The mode-switch and preview helpers (`_setRunnerOneRMMode`/`_setAdd1RMMode`,
`_updateRunnerEpleyPreview`/`_updateAdd1RMEpleyPreview`) are byte-level near-duplicates, which is the
likely reason the third writer was missed when `exercise_id` was introduced.

## Also: a falsy-zero divergence between kg and lb

`app-runner.js:722`:

```js
const wPlaceholder = oneRMPh || (prev?.weight_kg != null ? weightToPref(prev.weight_kg) : '') || '—'
```

`weightToPref` returns the NUMBER `0` in kg (falsy → falls through to `'—'`) and the STRING `"0"` in lb
(truthy → shows `0`). A previous 0 kg set ghosts differently by unit preference. Cosmetic, but the
identical shape already fixed and documented in `_setDetailsLine` (`app-progress.js:1234-1237`) — sibling
site, unfixed.
