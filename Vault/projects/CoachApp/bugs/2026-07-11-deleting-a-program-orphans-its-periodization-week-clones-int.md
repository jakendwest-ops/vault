---
id: 2026-07-11-deleting-a-program-orphans-its-periodization-week-clones-int
status: open
priority: high
reported: 2026-07-11
---

# deleting a program orphans its periodization week-clones into the reusable template pool

**DECISION NEEDED — deleting a program orphans its periodization week-clones into the reusable template pool.** The phase cascade sets `generated_from_phase_id = NULL`, so a "Bench Press — W2" clone loses the only column marking it a derivative. Options: FK CASCADE, or have `deleteProgram` sweep clones first.
