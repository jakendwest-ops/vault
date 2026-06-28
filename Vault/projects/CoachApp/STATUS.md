# CoachApp — STATUS
_Last updated: 2026-06-27_

---

## Live state

**App version:** v134 (app.js)
**Hosting:** GitHub Pages — https://jakendwest-ops.github.io/coachapp
**CSS version:** v=3 (main.css)
**Last push:** 645528d — PTHub v1 full app build (unrelated to CoachApp)
**Last CoachApp push:** Committed locally, not yet pushed to master

---

## What's working (verified)

- Auth (login / signup / invite accept / session persistence)
- PT dashboard — stats row, recent activity, compliance cards, goals due
- Client management — list, add, profile tabs, edit, invite, resend
- Workout templates — create / edit / delete / add exercises / log
  - Template edit/delete: coach_id guard + row-count check prevents silent permission failures
- Workout runner — set-by-set flow, AMRAP, rest timer; compact input row
  - **Timed sets** — Duration (mm:ss) field in template builder; duration log field in runner
  - **Unilateral L/R logging** — separate Left/Right columns for weight+reps in runner
  - Set X of Y visible in header; reps/weight target visible; PT notes always shown; client notes textarea; Edit button on logged sets
- **Skip rest bug fixed** — "Resting" toast now disappears correctly after clicking Skip
- Last session strip — persistent strip above stats bar in runner
- Client dashboard — goals, weight, upcoming, PBs, recent sessions, active program card
- Programs — create / edit / delete programs, phases, assign template to day, assign to client
- Client "My Programs" accordion — phase expand → session rows with exercise count → tap session → exercise detail; falls back to flat list if no program assigned
- Client session detail — session rows expandable to show exercise list (regression fixed)
- Calendar — monthly grid, add/delete events; client calendar shows program workouts
- Weight tracking — log, chart, stats row
- Performance / PBs — log, best-per-exercise, chart
- Check-ins — weekly check-in form, history
- Role-specific nav — PT gets 6 items; client gets 4
- My Progress page (client) — 4 tabs: Body Weight, Strength, Cardio, Personal Bests
- View switcher — PT ↔ client via sidebar (desktop) + mobile pill
- Pre-push hook — 9-item bug scan
- Playwright suite — runner + client session flows covered (15 tests, all green)
  - Skip rest test: asserts overlay gone, "Resting" text absent, LOG button visible
  - Client accordion tests: phase expand, session detail expand (conditional if program assigned)
- RLS — coach-scoped + client-scoped (SELECT + INSERT/UPDATE) + (SELECT auth.uid()) optimisation
- hello-claude + /save rituals active

---

## In progress / known gaps

- **`client_notes` column not yet in production.** Jake must run: `ALTER TABLE workout_log_exercises ADD COLUMN IF NOT EXISTS client_notes text;` Client notes written by runner will silently fail until this is done.
- **Timed sets + unilateral UI** — built and Playwright-covered but not verified on live site. Needs smoke test.
- **My Programs accordion** — Playwright tests are conditional (skip if test client has no program). Test client may not have a program assigned, making these tests no-ops. A more robust test would require assigning a program to the Playwright test client.
- **My Progress Strength tab** — uses `workout_logs!inner` join with PostgREST. May fail on live if filter syntax behaves differently from preview. Needs live test with real data.
- **Performance logs RLS** — client-scoped SELECT policy for `performance_logs` unconfirmed in production.
- **Program builder desktop layout** — at wider viewports cards may not span full main-content width.
- **Weekly check-in notification** — always shows "Due" if > 7 days; no dismiss until submitted.

---

## Continuity block

### Root cause to watch — modal rendering
Every instance of the "two-column layout" bug traced to:
1. Static modal HTML embedded inside `el.innerHTML` (inside `#main-content`)
2. Using `class="modal-box"` instead of `class="modal"`

**Pattern**: all modals must be dynamically created with `document.createElement` and appended to `document.body`. See `showAssignProgramModal` as the canonical pattern.

### Dynamic nav — event delegation
`renderNav(role)` rewrites `.sidebar-nav` and `.bottom-nav` innerHTML on every call. Click handlers use event delegation on the parent containers.

### `_runner` global structure
`{ clientId, name, date, exercises, exIdx, startTime, _timerInterval, _restInterval, _afterRest, lastSession }`
Each exercise: `{ name, type, targetSets, targetReps, targetWeight, loggedSets, bodyweight, sets_json, notes, oneRM, clientNotes }`

### `sets_json` flags
`s.timed` → duration field; `s.unilateral` → L/R split; `s.bodyweight` → BW display

### `_afterRest` pattern
Set for between-exercise rests: `() => { _runner.exIdx = nextExIdx; renderRunner() }`. Not set for between-sets rests — `skipRestTimer()` calls `renderRunner()` directly in that case.

### fetchRunnerLastSession — two-query pattern
Fetches last 20 `workout_logs` by `client_id`, then fetches `workout_log_exercises` filtered by `exercise_name` + `in(log_id, [...])`. Column names: `log_id`, `weight_kg`, `reps_achieved`, `set_number`.

### Preview discipline
After any desktop-width test, immediately resize back to 480×844.

### Cache busting
app.js is at `?v=134`. Next commit that changes app.js must bump to `?v=135`.

---

## Open to-dos for Jake

| Action | Priority |
|---|---|
| Run SQL migration: `ALTER TABLE workout_log_exercises ADD COLUMN IF NOT EXISTS client_notes text;` | **HIGH — blocks client notes feature** |
| Push latest CoachApp commits to master (GitHub Pages) | High |
| Test My Progress page on live site — Strength tab (PostgREST join) and Personal Bests (RLS on performance_logs) | Medium |
| Assign a program to the Playwright test client account so accordion tests are not no-ops | Medium |
| Update roadmap.md — client-facing features section now mostly Done | Low |

---

## Build history snapshot

| Version | What shipped |
|---|---|
| v134 | Timed sets, unilateral L/R logging, client My Programs accordion, runner redesign (Set X of Y, target display, PT notes, client notes, Edit button), skip rest bug fix, client session detail fix |
| v133 | 12-week Hyrox program build, phase structure, 44 session templates |
| v110 | Role-specific nav; compact runner input; last session strip; My Progress page (4 tabs); template edit/delete guard |
| v95 | Fix modal layout bug — apc-modal converted to dynamic body-level creation |
| v94 | Programs full build — phases, assign template to day, assign to client, client dashboard weekly schedule card |
| v80 | Post-save navigation fix (client lands on Workouts page) |
