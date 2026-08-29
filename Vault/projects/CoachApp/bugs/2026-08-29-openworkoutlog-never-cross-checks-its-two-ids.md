---
id: 2026-08-29-openworkoutlog-never-cross-checks-its-two-ids
status: open
priority: medium
reported: 2026-08-29
status_detail: "openWorkoutLog(logId, clientId) takes two independent ids and never compares clientId to logRow.client_id. The separate clientId drives the 'Last time' comparison query, the Delete button's argument and backToClientWorkouts. A coach passing client A's log id with client B's client id renders A's session with B's history as the baseline. Both ids are RLS-bounded to the caller's tenant, so this is within-tenant mis-attribution of health data, not a cross-tenant leak."
---

# `openWorkoutLog` takes two ids and never checks they refer to the same client

**js/app-runner.js:3027**, fetch at `:3038-3042`, consumers at `:3059-3062`, `:3094`, `:3104` —
found by the weekly full-file review (Agent A).

`logRow` is fetched `.eq('id', logId)` with no ownership anchor, and the separate `clientId` parameter
is **never compared to `logRow.client_id`** — yet it drives the "Last time" comparison query
(`:3061` `.eq('client_id', clientId)`), the Delete button's argument (`:3104`), and
`backToClientWorkouts` (`:3094`).

**Concrete:** a coach calling `openWorkoutLog('<client A's log id>', '<client B's client id>')` renders
client A's session with **client B's** training history as the "Last time" baseline and volume delta.

**Honest severity:** both ids are RLS-bounded to the caller's own tenant, so this is **within-tenant
mis-attribution of health data**, not a cross-tenant leak. It matters because a PT reading a wrong
baseline makes a wrong programming decision, and nothing on screen says the comparison is from a
different person.

**Fix:** `if (clientId && logRow?.client_id !== clientId) return` after the fetch — or better, derive
`clientId` from `logRow.client_id` and drop the parameter entirely, which removes the class rather than
guarding it.

**Closes when:** the two ids can no longer disagree (parameter dropped, or a check refuses the mismatch),
proven by a spec that passes a mismatched pair and asserts no foreign history is rendered.
