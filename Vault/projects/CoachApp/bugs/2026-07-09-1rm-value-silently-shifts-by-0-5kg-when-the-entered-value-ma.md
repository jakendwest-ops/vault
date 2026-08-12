---
id: 2026-07-09-1rm-value-silently-shifts-by-0-5kg-when-the-entered-value-ma
status: open
priority: high
reported: 2026-07-09
reported_detail: re-checked 2026-08-02
---

# 1RM value silently shifts by 0.5kg when the entered value matches an existing entry (e.g

**1RM value silently shifts by 0.5kg when the entered value matches an existing entry** (e.g. entering 200kg when another exercise is already 200kg saves as 199.5kg). **Re-investigated 2026-08-02, same conclusion reached independently**: traced `save1RM` (the actual save path, app-progress.js) end to end — `weightFromPref` passes a kg-preference value straight through with no rounding, and there's no dedup/cross-exercise logic anywhere in the insert/update path. Also checked the display formatter (`fmtWeight` via `latest.one_rm_kg`) — no rounding there either. Nothing in the code explains this mechanism. Needs a live repro (devtools open, watch the actual network request/response) before any further attempt — reading the code twice hasn't found a lead. NOT the %1RM-target rounding fixed 2026-07-10.
