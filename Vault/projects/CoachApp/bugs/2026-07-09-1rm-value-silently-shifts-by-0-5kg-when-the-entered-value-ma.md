---
id: 2026-07-09-1rm-value-silently-shifts-by-0-5kg-when-the-entered-value-ma
status: open
priority: high
reported: 2026-07-09
reported_detail: re-checked 2026-08-02
---

# 1RM value silently shifts by 0.5kg when the entered value matches an existing entry (e.g

**1RM value silently shifts by 0.5kg when the entered value matches an existing entry** (e.g. entering 200kg when another exercise is already 200kg saves as 199.5kg). **Re-investigated 2026-08-02, same conclusion reached independently**: traced `save1RM` (the actual save path, app-progress.js) end to end — `weightFromPref` passes a kg-preference value straight through with no rounding, and there's no dedup/cross-exercise logic anywhere in the insert/update path. Also checked the display formatter (`fmtWeight` via `latest.one_rm_kg`) — no rounding there either. Nothing in the code explains this mechanism. Needs a live repro (devtools open, watch the actual network request/response) before any further attempt — reading the code twice hasn't found a lead. NOT the %1RM-target rounding fixed 2026-07-10.

## INVESTIGATED 2026-08-26 — two real bugs found, but NEITHER explains the 0.5. Row STAYS OPEN.

This row demanded a live repro before any further attempt. Done. Findings, in order of certainty:

**1. Both prior investigations traced the wrong writer.** They followed `save1RM`. There are FIVE
`client_1rms` write paths; the grid (`saveOneRMGrid`, app-progress.js:119) is a different one and is
the surface where “another exercise is already 200” actually happens — you copy the number you can see.

**2. CONFIRMED + FIXED (different bug):** in lb, the display is a lossy proxy — 200 kg paints as
“440.9”, typing that back stores 199.99, and the next render rounds it to “200” so the drift is
invisible. Filed and fixed as
[[2026-08-26-lb-display-value-saved-back-corrupts-the-stored-kg]] (d320220, 7 sites).

**3. It does NOT explain this row.** The lb round trip is bounded at **~0.023 kg** and cannot
produce 199.5. The only thing in this path that yields exactly 0.5 is `step="0.5"` on the grid input
(app-progress.js:234) plus a deliberate arrow/spinner press — confirmed: one ArrowDown from a painted
“200” gives exactly “199.5”. Tested the accidental route (mouse wheel over a focused input) and it did
**NOT** step the value — caveat: headless Chromium, real Chrome may differ and I could not test that.

**Deliberately not closed.** Forcing the mechanism I found to be the one Jake reported is the les-049
failure (a confidently wrong root cause, repeated before checking). Two real bugs were fixed; his
symptom is still unexplained.

**The one question that would settle it:** was the weight unit set to **lb**, and do you remember
using the up/down arrows or the field’s spinner?
  - lb + no arrows → the fixed bug is very likely what you saw, and “0.5” was an approximation.
  - kg + arrows → it is the step, and the fix is to remove `step="0.5"` from a field where a stray
    keypress silently rewrites training data.
  - kg + no arrows → **both findings are the wrong track** and this needs a fresh investigation.
