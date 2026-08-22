---
id: 2026-08-22-client-1rms-write-class-still-has-two-unguarded-siblings
status: open
priority: medium
reported: 2026-08-22
status_detail: "Found by multi-agent-review (Agent B) 2026-08-22, immediately after I declared this write class closed in commit 8d389e7. The class has 12 members; I guarded 10 and the code comment says the inventory is complete. Not a solo break — these are simply still unguarded."
---

# The caller-supplied-clientId write class has two siblings I missed — and the comment claims it is closed

`js/app-core.js:734-736` inventories the class as *"every write keyed on a caller-supplied clientId —
client_1rms, performance_logs, weight_logs … **Ten** such writes across app-progress.js and
app-runner.js."* Two `client_1rms` inserts of exactly that shape are unguarded:

- **`saveOneRMGrid(clientId)`** — `js/app-progress.js:37`, insert at `:110`. Rendered as
  `onclick="saveOneRMGrid('${clientId}')"` at **app-progress.js:255** — inside `renderClient1RMs`, the
  *same render function* that emits the now-guarded `save1RM('${clientId}')` (`:324`) and
  `delete1RM('${latest.id}','${clientId}')` (`:231`). Same file, same table, same inline-onclick shape,
  three lines apart. Skipped.
- **`_saveMissingOneRMEntries(clientId)`** — `js/app-programs.js:615`, insert at `:624`. Called from
  `app-programs.js:365` and `:757` (the programme assign flow). **app-programs.js is not in the
  comment's inventory at all.**

## Why this one stings

The standing behaviour is *"FIX THE CLASS, NOT THE INSTANCE — grep for every other function doing the
same job, COUNT them."* I ran that sweep, found ten, fixed ten, and then wrote a comment asserting the
inventory was complete. It was not. The count came from the ledger row's list rather than from my own
grep of the codebase — I inherited someone else's enumeration and presented it as a swept class.

The same review pass caught the identical pattern one layer up: a "fix the class" commit that fixed 2 of
7 sites. Two instances of the same meta-failure in one day.

## Not urgent

Neither is a solo break and neither is newly exposed — they are simply still in the state every one of
these writes was in before 2026-08-21. RLS backstops the family (proven
`tests/audit-ownership-anchors-rls-2026-08-12.spec.js`). The defect is the false claim of completeness
as much as the gap.

**Closes when:** both route through `_verifyClientAccess`, the app-core comment's inventory is corrected
to name app-programs.js and the real count, and a test proves each refuses a foreign clientId.
