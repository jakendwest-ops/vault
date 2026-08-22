---
id: 2026-08-22-zero-sessions-guard-exists-twice-only-one-copy-is-tested
status: open
priority: medium
reported: 2026-08-22
status_detail: "Found 2026-08-22 while establishing red-before for the 2026-07-13 zero-sessions crash. Found by accident: I neutered the wrong copy, the test still passed, and checking why revealed a second untested copy."
---

# The zero-sessions crash guard exists in TWO places; only one has a test

The 2026-07-13 critical — *"a phase with zero sessions kills the coach's entire client Programs tab"* —
was fixed with a `!weekNums.length` empty-state guard. That guard exists **twice**, byte-identical:

- `js/app-programs.js:140` — inside `renderClientPrograms` (coach → client profile → Programs tab).
  **Covered** by `regression-2026-07-13.spec.js`; red-before established 2026-08-22.
- `js/app-workouts.js:747` — the sibling render. **No test.** Neutering it changes nothing that any
  spec observes.

## How it was found

By accident, which is the uncomfortable part. Establishing red-before for the critical row, I neutered
`app-workouts.js:747` first, the test still passed, and I was one step from reporting the test as
decorative. Checking *which module owns `renderClientPrograms`* showed I had simply broken an
unobserved code path.

So the near-miss was mine — but the underlying fact stands: **one of the two copies of a fix for a
CRITICAL live crash has no coverage at all.**

## Why it matters

This is the documented `deletePhaseWeek` / `_cleanupPhaseWeeksBeyond` shape (2026-07-10): a fix landed
where the bug was *found*, its sibling was missed, and the sibling destroyed real Week-1 workouts for a
day. Here both copies got the fix — but only one got the test, so only one is defended against
regression.

The guard is a duplicated literal, which is the other half of the problem: the empty-state markup is
copy-pasted rather than shared, so the two can drift.

**Closes when:** either the `app-workouts.js:747` path gets its own red-before/green-after test, or the
two copies are collapsed into one shared helper that the existing test already covers.
