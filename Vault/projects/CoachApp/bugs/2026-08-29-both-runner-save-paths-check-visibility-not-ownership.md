---
id: 2026-08-29-both-runner-save-paths-check-visibility-not-ownership
status: open
priority: high
reported: 2026-08-29
status_detail: "saveRunnerSession and saveWorkoutSession prove the clients row is READABLE, not that it is ours. _verifyClientAccess asserts the latter and fails closed. Counted: 2 of the 4 client-scoped writers in app-runner.js use the strong check; the two save paths do not - and the helper was explicitly modelled on one of them."
---

# Both runner save paths do a *visibility* check where the extracted helper does an *ownership* check

**js/app-runner.js:2345** (`saveRunnerSession`) and **js/app-runner.js:2927** (`saveWorkoutSession`).

    const { data: clientRecord } = await dbq('...:clientLookup',
      db.from('clients').select('coach_id').eq('id', clientId).single(), { showUserError: false })
    if (!clientRecord) { ...refuse... }
    const coachId = clientRecord.coach_id || currentUser.id

This proves the `clients` row is **readable**. It does not prove
`coach_id === currentUser.id || user_id === currentUser.id`. `_verifyClientAccess`
(`app-core.js:1105-1112`) asserts exactly that and fails closed on anything else.

**Counted, not impressionistic — 4 client-scoped writers in this one file, 2 use the strong check:**

| Writer | calls `_verifyClientAccess`? |
|---|---|
| `_savePostSessionOneRM` :2543 | yes |
| `saveRunnerOneRM` :2663 | yes |
| `saveRunnerSession` :2345 | **no** |
| `saveWorkoutSession` :2927 | **no** |

**The asymmetry is itself the evidence.** `app-core.js:1093-1095` records that `_verifyClientAccess` was
*"deliberately modelled on saveRunnerSession (app-runner.js:2345)"*. The two originals were never upgraded
to the helper they inspired, while the two 1RM writers in the same file were.

**Concrete failure:** any `clients` SELECT policy broader than "mine" — a co-client view, a support role,
a future gym/team feature — and both save paths pass their own guard and insert `workout_logs` for that
client. Today the only thing refusing that is `workout_logs_insert_owned_client_only`
(`scripts/fix-workout-logs-insert-policy-2026-07-30.sql:67`) — the database, not the app. That policy
exists *because* a cross-tenant insert here was confirmed exploitable on 2026-07-30. The app-level guard
added alongside it is weaker than the helper later extracted from it.

**Fix:** call `_verifyClientAccess` in both, keeping the existing `coach_id` derivation from the same fetch.

**Closes when:** both call `_verifyClientAccess`, AND a spec drives each with a client id the caller can
READ but does not OWN — RED before, GREEN after. Note the other half:
[[feedback_guard_risk_is_refusing_the_legitimate_user]] — solo (coach_id NULL, user_id = uid) must still
pass, so the spec must assert the legitimate case too, not only the refusal.
