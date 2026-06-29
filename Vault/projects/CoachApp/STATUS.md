# CoachApp — STATUS
_Last updated: 2026-06-29_

---

## Live state

**App version:** v157 (app.js)
**Hosting:** GitHub Pages — https://jakendwest-ops.github.io/coachapp
**CSS version:** v=3 (main.css)
**Last push:** 401f309 — v157 settings smoke tests + delete modal position fix (pushed 2026-06-29, CI green)
**Supabase project:** avilxuiacmtgeoxxhfhc (eu-west-1, Ireland)

---

## What's working (verified)

- Auth — login / signup (with consent checkbox) / invite accept / session persistence
- PT dashboard — stats row, recent activity, compliance cards, goals due; business name in subtitle when branding set
- Client management — list, add, profile tabs (Overview / Goals / Workouts / Weight / Performance / Programs / Photos / 1RMs), edit, invite, resend
- Workout templates — create / edit / delete / add exercises / log
- Set form — AMRAP, Unilateral, Timed, Reps, Weight, %1RM, Rest, RPE/RIR, Tempo, Notes
- Cardio set form — Pace, HR Zone, Rest, Stroke rate, Duration, Distance
- Template card set preview — correct for all set types including timed (1:30 not 90 reps)
- Workout runner — set-by-set flow, timed sets, unilateral L/R, rest timer, skip rest, last session strip
- Edit logged sets in runner; PT notes always shown; client notes textarea
- Runner save lands on correct page per role (client → Workouts, PT → client profile)
- Client dashboard — goals, weight, upcoming events, PBs, recent sessions, active program card; PT branding header when set
- Programs — create / edit / delete / phases / assign template to day / assign to client / edit start date / remove
- Edit sessions directly from Programs page (backFn context pattern)
- Program phases: client plan clones have program_id: null (bug fixed 2026-06-28)
- Client programs accordion — phase → day → SESSION N/M → exercise list → Start button
- Calendar — monthly grid, add/delete events; client calendar shows program workouts
- Weight tracking — log, chart, stats row
- Performance / PBs — log, best-per-exercise, chart, delete
- Check-ins — weekly form, history
- Goals — create, edit, milestones, check-ins, due-soon on PT dashboard
- My Progress (client) — 4 tabs: Body Weight, Strength, Cardio, Personal Bests
- View switcher — PT ↔ client for master account (sidebar desktop + mobile pill)
- Exercise library — add / edit / delete / muscle groups / library dropdown in template exercise editor
- Programs builder — Hyrox Hero (12-week, 4 templates × 12 phases = 48 master templates)
- **Branding** — PT logo upload (private logos bucket, signed URLs), business name, sidebar, PT dashboard, client dashboard
- **Security/GDPR** — private storage buckets, PII-free logs, consent checkbox, data export, delete account (delete_current_user RPC)
- Pre-push hook — 10-check bug scan (JS syntax, column names, query scoping, cache bust, no alert, no hardcoded IDs, no set_type, no swallowed errors, no bare clearInterval, no PII in logs, no timed set guard bypass, no duplicate functions)
- GitHub Actions CI — mirrors pre-push hook
- Playwright E2E — 31 tests covering runner, client session, auth flows, settings (timeout bumped to 20s)
- Delete account — custom modal, typed DELETE confirmation, proper error handling, anchored to top of viewport (no browser dialogs, no clipping when page scrolled)
- XSS protection — `escapeHtml()` applied to all coach-controlled strings in innerHTML
- Playwright E2E — 31 tests (5 new settings smoke tests: sections render, delete modal open/cancel/validation, download data)

---

## In progress / known gaps

- **Privacy policy page** — consent checkbox links to `#`; real policy needed before first beta invite
- **Invite email logo** — PT branding not yet included in invite email HTML (Edge Function not updated)
- **Signed URLs for progress-photos expire after 1hr** — images break in long-lived tabs; acceptable for now
- **My Programs accordion Playwright tests** — conditional (skip if test client has no program). Test client may not have program assigned.
- **My Progress Strength tab** — PostgREST `!inner` join; not verified on live with real data
- **Performance logs RLS** — client-scoped SELECT policy for `performance_logs` unconfirmed in production
- **Program builder desktop layout** — cards may not span full width at wider viewports
- **Weekly check-in notification** — always shows Due if >7 days; no dismiss until submitted

---

## Continuity block

### Modal pattern — body-level only
All modals: `document.createElement` → `document.body.appendChild`. Never embed in `el.innerHTML`. See `showAssignProgramModal` as canonical pattern. `class="modal-overlay"` wraps `class="modal-box"`.

### `program_id` constraint
Templates with `program_id` set are hidden from flat Workouts list. Client plan clones: `program_id: null`, `client_id: set`. Master templates: `program_id: set`, `client_id: null`. Standalone: both null.

### Timed sets dual format
`s.duration` = `mm:ss` string (from editor). `s.repsMin` = seconds integer string (from programmatic build). Render must handle both. Pattern: `parseRest(s.duration) || parseInt(s.repsMin)`.

### Dynamic nav — event delegation
`renderNav(role)` rewrites `.sidebar-nav` and `.bottom-nav` innerHTML on every call. Click handlers use event delegation on parent containers — never inline onclick on nav items.

### `_runner` global structure
`{ clientId, name, date, exercises, exIdx, startTime, _timerInterval, _restInterval, _afterRest, lastSession }`
Each exercise: `{ name, type, targetSets, targetReps, targetWeight, loggedSets, bodyweight, sets_json, notes, oneRM, clientNotes }`

### `sets_json` flags
`s.timed` → duration field; `s.unilateral` → L/R split; `s.bodyweight` → BW display; `s.amrap` → AMRAP mode

### `_afterRest` pattern
Between-exercise rests: `() => { _runner.exIdx = nextExIdx; renderRunner() }`. Between-set rests: `skipRestTimer()` calls `renderRunner()` directly.

### `_templateCtx` / `_templateGoBack`
Back-nav context for template editor. Always set `backFn` when opening template from non-standard location. `_templateGoBack` checks: `backFn` → `clientId` → `backTo` → default `'workouts'`.

### Save functions own no navigation
`save*` functions save only. Navigation is role-aware at the call site.

### Master account
`window._masterAccount = true` when user has both coach profile and client record. `currentProfile.role` flips between `'coach'` and `'client'` via `switchView()`. `localStorage._activeView` persists choice.

### Storage — signed URLs
Private buckets: logos (604800s = 7 days), progress-photos (3600s = 1hr). Never `getPublicUrl`. Use `createSignedUrl` (single) or `createSignedUrls` (batch).

### Cache busting
app.js is at `?v=157`. Next commit that changes app.js must bump to `?v=158`.

---

## Open to-dos for Jake

| Action | Priority |
|---|---|
| Write and publish privacy policy page; update consent checkbox href | **High — blocks beta** |
| Live smoke test on GitHub Pages — branding, data export, delete account modal (v157 — modal position fixed) | **High** |
| Update invite-client Edge Function to include PT logo in invite email HTML | Medium |
| Test My Progress page on live — Strength tab and Personal Bests (RLS on performance_logs) | Medium |
| Assign a program to the Playwright test client so accordion tests are not no-ops | Medium |
| ICO breach notification procedure — document before beta | Medium |

---

## Build history snapshot

| Version | What shipped |
|---|---|
| v157 | Settings smoke tests (5 tests); delete modal anchored to viewport top when page scrolled |
| v156 | Delete modal position fix — align-items:flex-start so modal not clipped when triggered scrolled |
| v155 | XSS fix — escapeHtml() on all businessName innerHTML injection points; downloadMyData error handling |
| v154 | deleteAccount custom modal — typed DELETE confirmation, replaces confirm()/prompt() |
| v153 | Code audit fixes — fire-and-forget DB write, dead code, orphaned unscoped function |
| v152 | Security/GDPR hardening — progress-photos private, consent checkbox, data export, delete account |
| v151 | PT branding — logo upload, business name, sidebar/PT dashboard/client dashboard display |
| v150 | Edit sessions from Programs page; exercise library dropdown in edit modal; program_id clone bug fix; timed set render fix (1:30 not 90 reps); PII stripped from 16 log sites; pre-push checks added |
| v134 | Timed sets, unilateral L/R logging, client My Programs accordion, runner redesign |
| v133 | 12-week Hyrox Hero program build (48 master templates) |
| v110 | Role-specific nav; compact runner input; last session strip; My Progress page |
| v95 | Fix modal layout bug — modals converted to dynamic body-level creation |
| v94 | Programs full build — phases, assign template to day, assign to client |
| v80 | Post-save navigation fix (client lands on Workouts page, not PT profile) |
