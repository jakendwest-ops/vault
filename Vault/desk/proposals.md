# Improvement proposals

_Evidence-backed proposals for structural OS changes. Filed here; never re-proposed after decline. Format: ## YYYY-MM-DD — title, then Evidence, Proposal, Expected gain._

---

## Declined
_(none yet)_

---

## Open

### 2026-06-15 — Bank the existing codebase pattern index at session start

**Evidence:** During the RTL time input task, `perfRtlTime()` was written from scratch as a new function. The existing `wbRtlTime()` was in the codebase and followed the same 4-digit pattern — a 6-digit variant could have been derived from it in seconds. Neither STATUS nor CRITICAL flagged the function's existence, so it was re-discovered by reading the file.

**Proposal:** Add a "Key functions" section to PTHub's CRITICAL.md listing non-obvious utility functions (RTL handlers, flush patterns, date helpers) with one-line signatures. This is a small volatile set — maybe 10 entries — that would cut "find the pattern" reads across sessions.

**Expected gain:** Faster implementation of variants on existing patterns; fewer full-file reads mid-session. The test: future Vision shouldn't need to grep for `wbRtlTime` to know it exists.
