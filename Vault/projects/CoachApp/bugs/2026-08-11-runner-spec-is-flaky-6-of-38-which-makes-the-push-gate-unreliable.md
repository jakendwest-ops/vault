---
id: 2026-08-11-runner-spec-is-flaky-6-of-38-which-makes-the-push-gate-unreliable
status: open
priority: medium
reported: 2026-08-11
---

# runner.spec.js is flaky (6 of 38), so checks.sh blocks pushes at random

Measured 2026-08-11 across three runs:

- Full suite: 384 passed, **0 failed**, 8 flaky
- `runner.spec.js` alone: 32 passed, **0 failed**, 6 flaky
- `checks.sh`: 1 "failed" (`runner.spec.js:623`) — which passes alone AND in a full runner.spec.js run

Different tests flake each run and all pass on retry, so this is not a code regression — a regression
would fail the SAME test every time. It is the fixture-borrowing pattern already recorded as a lesson:
tests that `.first()` against shared data rather than owning their fixtures.

**Why it matters:** `checks.sh` is the pre-push gate. A gate that fails at random either blocks good
pushes or trains you to ignore it — and an ignored gate is worse than no gate.

Not caused by this session's work; surfaced by running the suite many times in one day.
NOT YET FIXED — needs the flaky tests given their own fixtures.
