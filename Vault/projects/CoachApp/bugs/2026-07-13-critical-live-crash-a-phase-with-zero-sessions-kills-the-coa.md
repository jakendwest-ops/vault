---
id: 2026-07-13-critical-live-crash-a-phase-with-zero-sessions-kills-the-coa
status: fixed-awaiting-jake
priority: critical
reported: 2026-07-13
status_detail: "fixed — awaiting Jake"
---

# CRITICAL — LIVE CRASH: a phase with zero sessions kills the coach entire client Programs tab. app-programs.js

🔴 **CRITICAL — LIVE CRASH: a phase with zero sessions kills the coach entire client Programs tab.** `app-programs.js:129` does `renderDays(weekMap[weekNums[0]])` with **no `!weekNums.length` guard** → `sessions.forEach` on undefined → TypeError thrown *while building the template literal*, so the whole `el.innerHTML` assignment never lands: **every phase of every assigned program for that client vanishes; the tab is stuck on Loading…**. Repro is the NORMAL build order: add a 3rd phase, assign the program, open Client → Programs before populating it. **This exact bug was fixed on 2026-07-10 (b79c152) in the verbatim twin `app-workouts.js:352`, which HAS the guard.** The fix never got ported. 5th instance of fix-the-class-not-the-instance.
