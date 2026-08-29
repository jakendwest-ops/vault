---
id: 2026-08-17-effectivecoachidforclient-swallows-an-rls-denial
status: open
priority: medium
reported: 2026-08-17
status_detail: "Found independently by two review agents. Same shape as the pattern hardened out of saveRunnerSession on 2026-07-30, which a live cross-tenant probe confirmed exploitable there."
---

# `_effectiveCoachIdForClient` falls back to `currentUser.id` on a lookup FAILURE

`js/app-workouts.js:1841-1844`:

```js
const { data: clientRow } = await db.from('clients').select('coach_id, user_id').eq('id', clientId).single()
return clientRow?.coach_id || clientRow?.user_id || currentUser.id
```

`|| user_id` exists for SOLO, whose `coach_id` is legitimately NULL. But the trailing `|| currentUser.id`
makes an RLS-denied lookup (0 rows → `data` null) indistinguishable from a real solo record. Verbatim what
`saveRunnerSession` documents at `app-runner.js:2310-2324`:

> "MUST fail loud, not fall back to `currentUser.id` ... a clientId RLS denies (0 rows) used to silently
> do the exact same fallback."

The guard was added to two call sites on 2026-07-30 and never applied to this shared resolver. Same shape
at `app-workouts.js:1704-1705` and `app-runner.js:2774`.

## Blast radius — smaller than the runner case, not nil

Callers: `saveOneRMGrid` (`app-progress.js:98`), `showAdd1RMModal` (`:264`), `_reopenExercisePickerFor1RM`
(`:336`). The value only selects which exercise library is read/written, so the failure mode is "you see
your own library", not a cross-tenant write.

But `_resolveExerciseIdForSave` (`app-workouts.js:1849-1857`) ends in an INSERT into `exercises`, and its
`is_personal` comes from `currentProfile?.role === 'solo'` — so a foreign client's lift name can become a
permanent row in the current user's personal exercise library.

## Related

`js/app-progress.js:98-110` — a CLIENT-role user saving a 1RM against a Big-5 name not in their coach's
library issues an insert into `exercises` with `coach_id` = another user's uid.
`_resolveExerciseIdForSave`'s contract comment scopes it to "the Big 5 quick-start 1RM form", which was
coach-side only when written; `renderClient1RMs` is now the client/solo Personal Bests tab too. It
degrades safely IF the `exercises` INSERT policy is `coach_id = auth.uid()` — that policy is the only
thing stopping it, and it has not been verified behaviourally.

---

## 2026-08-29 — re-confirmed live by the weekly full-file review

Agent A checked it independently and found `js/app-workouts.js:1940-1943` **unchanged**:

    const { data: clientRow } = await db.from('clients').select('coach_id, user_id').eq('id', clientId).single()
    return clientRow?.coach_id || clientRow?.user_id || currentUser.id

An RLS denial still becomes "it's mine". `app-core.js:1093-1096` explicitly names this function as the
**anti-model** that `_verifyClientAccess` was written against, and cites this row by id.

**One live caller in scope:** `renderClientWorkoutsPage:658`, feeding `:659`'s `.eq('coach_id', coachId)`
template fallback. That path is **read-only**, so a swallowed denial yields a wrong or empty template list
rather than a leak — which is why this stayed medium rather than escalating.

It is now the **last site in the reviewed files that treats an unreadable row as ownership.** Its natural
companion is `2026-08-29-both-runner-save-paths-check-visibility-not-ownership`: both are the same
mistake — inferring ownership from readability — and both are fixed by routing through
`_verifyClientAccess`. **Do them together**; fixing one and not the other is
[[feedback_fix_the_class_not_the_instance]] a third time in one file.

**Closes when:** `_effectiveCoachIdForClient` either propagates the denial or routes through
`_verifyClientAccess`, proven by a spec where the `clients` read is refused and the function does NOT
return `currentUser.id`.
