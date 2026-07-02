# CoachApp — STATUS
_Last updated: 2026-07-02 (session 8)_

---

## Live state

**App version:** app-core/dashboard/clients v=1 · app-programs v=5 · app-calendar-goals v=2 · app-workouts v=6 · app-runner v=4 · app-progress v=3
**Hosting:** GitHub Pages — https://jakendwest-ops.github.io/coachapp
**CSS version:** v=3 (main.css)
**Last push:** 0a0f89f — collapsible session history (client + PT), batched phase-workout query, delete-nav target fixes, dead "group by program" code removed. Found as uncommitted local work at session start (unknown prior authorship — no LOG entry existed for it); verified this session via Playwright (48/48) + 3-agent review before push. Pre-push hook: all checks + 19 smoke tests green. (Previous: 7eabcc0 — removed 8 project-level `.claude/skills/` duplicates, pushed 2026-07-01.)
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
- Client dashboard — hero "Up next" card (program + phase + week), goals, weight, upcoming events, PBs, recent sessions; PT branding header when set
- PT dashboard — hero card, stats strip, two-column grid, activity feed (coach-scoped, no solo bleed), compliance cards, goals due
- Solo dashboard — hero card, stats strip (hidden on mobile), two-column grid
- Sudo/impersonation mode — "View as" button on client list (email-gated), amber banner, exit restores PT view
- Progress tabs — Body Weight and Personal Bests tabs have add buttons; Cardio explains auto-population
- Programs — create / edit / delete / phases / assign template to day / assign to client / edit start date / remove
- Edit sessions directly from Programs page (backFn context pattern)
- Program phases: client plan clones have program_id: null (bug fixed 2026-06-28)
- Client programs accordion — phase → day → SESSION N/M → exercise list → Start button
- Calendar — monthly grid, add/delete events; client calendar shows program workouts
- Session history (client Workouts page + PT client-profile Workouts tab) — collapsed behind a toggle by default instead of always showing the full list (2026-07-02)
- Weight tracking — log, chart, stats row
- Performance / PBs — log, best-per-exercise, chart, delete
- Check-ins — weekly form, history
- Goals — create, edit, milestones, check-ins, due-soon on PT dashboard
- My Progress (client) — 4 tabs: Body Weight, Strength, Cardio, Personal Bests
- View switcher — PT | Client | Personal three-pill for master account (sidebar desktop + mobile); Personal pill shown only when solo client record exists
- Personal (solo) account — Jake's client record severed from PT (coach_id = null); personal dashboard, solo nav, self-assign programs, Workouts shows program session accordion with Start buttons
- Exercise library — add / edit / delete / muscle groups / library dropdown in template exercise editor
- Programs builder — Hyrox Hero (12-week, 4 templates × 12 phases = 48 master templates)
- **Branding** — PT logo upload (private logos bucket, signed URLs), business name, sidebar, PT dashboard, client dashboard
- **Security/GDPR** — private storage buckets, PII-free logs, consent checkbox, data export, delete account (delete_current_user RPC)
- Pre-push hook — scans all 8 module files (updated 2026-06-30); JS syntax, column names, query scoping, cache bust, no alert, no hardcoded IDs, no set_type, no swallowed errors, no bare clearInterval, no PII in logs, no timed set guard bypass, no duplicate functions
- GitHub Actions CI — mirrors pre-push hook
- Playwright E2E — 47 tests (40 + 7 new in tests/programs.spec.js on 2026-07-01: periodization Linear/Undulating, 1RM assignment-check missing/have, inline grid render+search, race-guard, create-workout back-link); 19 smoke tests in pre-push hook
- **Codebase modularised** — app.js split into 8 modules: app-core, app-dashboard, app-programs, app-clients, app-calendar-goals, app-workouts, app-runner, app-progress
- Delete account — custom modal, typed DELETE confirmation, proper error handling, anchored to top of viewport (no browser dialogs, no clipping when page scrolled)
- XSS protection — `escapeHtml()` applied to all coach-controlled strings in innerHTML
- Session detail slide-in — right-side drawer showing exercises/sets/reps for a session; works in solo and client mode; `position:fixed;inset:0;z-index:1000` wrapper pattern
- `sudoAsClient()` server-side guard — in-function email check prevents DevTools exploitation by non-Jake users
- **DB security hardening** — OpenAPI schema mocked; `lock_created_at` BEFORE UPDATE triggers on 11 tables; `events` UPDATE USING clause fixed; REVOKE EXECUTE on 4 internal functions; `delete_current_user` search_path locked; max_rows 200; auto-expose new tables off; secure password change on; min password 8 chars; Security Advisor: 0 errors
- **Periodization (Linear / Undulating)** — phase-level %1RM automation; PT builds Week 1 once, picks a scheme, clicks "Generate weeks"; propagates to already-assigned clients automatically; idempotent regeneration; shrinking a phase's week count prunes orphaned generated weeks (2026-07-01)
- **1RM assignment-time check** — when assigning a program, shows which needed %1RM lifts the client is missing, with inline quick-fill (direct kg or Epley estimate); never blocks; covers both assign entry points + solo (2026-07-01)
- **Solo account write access to `client_1rms` and `client_programs`** — fixed a real gap where solo users had no write policy for saving 1RMs or removing/editing an assigned program (silently broken since solo accounts were built); 5 new RLS policies added (2026-07-01)
- **Inline assign-workout grid** — replaced the old "+ Assign workout" modal (day → session → template, one slot at a time) with an always-visible 7-day searchable grid on the phase card; picking a template assigns immediately, no modal, no separate day/session step. Search + exercise-preview hints per option. Picker excludes client-owned clones and periodization-generated week-clones (`generated_from_phase_id` column). Race-condition guard re-checks a slot is still free immediately before inserting. "Create new workout" now returns to the program via a working "Back to program" link instead of stranding the coach (2026-07-01)

---

## In progress / known gaps

- **Runner bug fixes (session history double-display, phantom "Set 5 of 4", beep timing, audio unlock)** — root-caused and coded this session, cache-bust bumped, but **not yet verified or pushed** — session ended mid-testing. See Open to-dos.
- **Privacy policy page** — ✅ Done (v158, pushed 2026-06-29) — `/privacy-policy.html` live, consent link updated
- **Personal account (solo view)** — ✅ Done (v158–v162, pushed 2026-06-29) — PT | Personal view switcher, solo dashboard, programs self-assign, Workouts session accordion, RLS policies added
- **Performance logs RLS** — ✅ Done (2026-06-29) — 5 clean policies confirmed via pg_policies query
- **Invite email logo** — PT branding not yet included in invite email HTML (Edge Function not updated)
- **Signed URLs for progress-photos expire after 1hr** — images break in long-lived tabs; acceptable for now
- **My Programs accordion Playwright tests** — conditional (skip if test client has no program). Test client may not have program assigned.
- **My Progress Strength tab** — PostgREST `!inner` join; not verified on live with real data
- **Program builder desktop layout** — cards may not span full width at wider viewports
- **Weekly check-in notification** — always shows Due if >7 days; no dismiss until submitted
- **Solo account Playwright tests** — ✅ 8 smoke tests added (v176); session detail slide-in smoke test added (v179); tests skip gracefully when E2E account has no solo client record

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

### Master account / solo account
`window._masterAccount = true` when coach user has any client record (coached or personal). `window._soloClientId` = personal client record ID (coach_id = null). `window._masterClientId` = coached client record ID (coach_id set). `currentProfile.role` cycles between `'coach'`, `'client'`, `'solo'` via `switchView()`. `localStorage._activeView` persists choice. Client pill only shown when `_masterClientId` set; Personal pill only when `_soloClientId` set.

### Storage — signed URLs
Private buckets: logos (604800s = 7 days), progress-photos (3600s = 1hr). Never `getPublicUrl`. Use `createSignedUrl` (single) or `createSignedUrls` (batch).

### Cache busting
Each of the 8 module files has its own independent `?v=N`. Any commit that changes a module file must bump that file's own version in index.html — bump only the files that changed, not all 8. Current: core/dashboard/clients v=1 · programs v=5 · calendar-goals v=2 · workouts v=6 · runner v=4 · progress v=3.

### Periodization — week_number / tier
`program_phases.periodization_type` (`'linear'`/`'undulating'`/null) + `periodization_config` (jsonb). `program_phase_workouts.week_number` (default 1) lets one phase hold distinct day/template assignments per week — phases that never use periodization just have week_number=1 rows, which the client calendar/workouts-page render as repeating every week (legacy behaviour, unchanged). `program_phase_workouts.tier` (`'heavy'`/`'moderate'`/`'light'`) is undulating-only, set per day-slot. `generatePhasePeriodization(phaseId, programId)` clones Week 1's templates into weeks 2..duration_weeks, recalculating only sets where `intensityMin`/`intensityMax` is set — everything else (reps, rest, tempo) is copied unchanged. Regeneration is idempotent (`_cleanupPhaseWeeksBeyond` deletes stale weeks + their templates, both master and any already-propagated client copies, before regenerating) — the same helper runs when a phase's `duration_weeks` is edited down. Client propagation: assigning a program clones week_number through (`_cloneTemplateForClient`/`_cloneProgramForClient`); if a client is *already* assigned when new weeks are generated, `generatePhasePeriodization` propagates to them too.

### 1RM assignment-time check
`_getProgramOneRMStatus(programId, clientId)` scans Week 1 only (sufficient — generated weeks reuse the same exercise names) for %1RM-tagged sets, diffs against `client_1rms` (trim+lowercase exact match — same limitation as the rest of the app; exercise_id-based matching is a deferred future decision, see roadmap). `_renderOneRMQuickEntry(idPrefix, name)` is the shared toggleable kg/Epley-estimate row component — parameterized by idPrefix so multiple rows coexist. `window._missingOneRMExercises` holds the current checklist's missing-name list (index-matched to `mor-N` DOM ids) between refresh and save. `_refreshMissingOneRMs` is token-guarded (`_oneRMRefreshToken`) so a stale in-flight refresh can't overwrite a newer one if the PT changes the client/program selection quickly. Wired into both assign entry points (`showAssignProgramModal`/`showAssignProgramToClientModal`) — 1RM entries must be saved *before* the modal is removed, not after (DOM reads fail silently on a detached modal).

---

## Open to-dos for Jake

| Action | Priority |
|---|---|
| **Runner bug fixes sitting uncommitted** — `app-runner.js` + `app-workouts.js` locally modified (set-counter cap, rest-timer render-order fix, 5s beep window, earlier audio unlock), cache-bust already bumped (workouts v7, runner v5) but **not yet run through Playwright, not code-reviewed, not pushed**. Session ended mid-verification. Must go through the full pre-push discipline before pushing — do not push as-is. | **High** |
| Live-device check the audio-unlock fix specifically once pushed — code-level fix applied but can't be confirmed by automated testing, only real playback on your phone | High |
| **Obsidian wiki — 5-part follow-up requested, not yet done:** (1) check the Web Clipper browser extension is actually configured correctly right now — needs live inspection, not just written guidance; (2) expand `guide-web-clipper` with concrete usage scenarios — branding/UI inspiration, competitor sites, review pages/Reddit threads about competitors; (3) a guide page on whether/how Obsidian+Claude can be set up to "continually improve themselves" — needs scoping with Jake first, no such automated mechanism exists yet; (4) clarify in `guide-new-project-setup` whether each new project gets its own CLAUDE.md or shares the one at the wiki root (current design: one shared CLAUDE.md for the whole wiki) — Jake's question suggests this isn't explicit enough yet; (5) a full re-read/quality pass on all 10 existing guide pages for a true-novice audience | **High** |
| Run /deploy-check before next beta invite | **High** |
| Tighten `tests/client-workout.spec.js` "session history" locator to be onclick-scoped like its sibling tests (currently text-only match — safe today, would break under Playwright strict mode if a second "Session history" button ever appears) | Low |
| **1RM exercise-name matching — exercise_id vs fuzzy string match** — big design decision, deferred to its own session (see roadmap.md). Today's build keeps name-matching in one shared helper so this swap is contained later. | Medium |
| **`deleteProgram()` doesn't clean up cloned workout_templates** — deleting a program cascades away phases/program_phase_workouts but orphans any templates cloned into it (periodization-generated or otherwise). Found via leftover E2E test debris, not a live bug on Jake's account yet, but will recur for any real program with generated weeks that gets deleted. Fix: before deleting the program, collect template_ids via program_phase_workouts and delete those too. | Medium |
| Create a solo client record on the E2E PT account so solo-account.spec.js tests stop skipping (all 9 tests skip against E2E account; only pass against Jake's account) | **High** |
| Assign a program to the Playwright test client (coachapp.e2e.client) so accordion tests are not no-ops. Confirmed wider this session: not one standalone workout template belonging to that client's coach has any exercises attached — blocked a live manual runner test entirely. Needs at least one real, populated template, not just a program assignment. | High |
| Verify iOS Safari slide-in fix on real device — `inset:0` replaced with explicit `top/right/bottom/left` in v180, pushed. Test on iPhone and close if working. | **High** |
| Run Rowing/Running/SkiErg DELETE SQL in Supabase (safety check first — see below) | High |
| Update invite-client Edge Function to include PT logo in invite email HTML | Medium |
| Test My Progress Strength tab on live with real data | Medium |
| ICO breach notification procedure — document before beta | Medium |
| Verify `workout_template_exercises` RLS policy exists (check pg_policies for that table) | Medium |
| Upgrade Supabase to Pro plan when ready for beta — unlocks leaked password protection (HaveIBeenPwned) | Low |

**Rowing/Running/SkiErg SQL (run in Supabase SQL editor):**
```sql
-- Step 1: safety check — confirm not in use
SELECT e.name, count(wte.id) as in_use
FROM public.exercises e
LEFT JOIN public.workout_template_exercises wte ON wte.exercise_id = e.id
WHERE e.name IN ('Rowing', 'Running', 'SkiErg')
GROUP BY e.name;
-- Step 2 (only if all counts = 0):
DELETE FROM public.exercises WHERE name IN ('Rowing', 'Running', 'SkiErg');
```

---

## Build history snapshot

| Version | What shipped |
|---|---|
| app-programs v=5 / workouts v=6 / runner v=4 / progress v=3 | Collapsible session history (client + PT), batched phase-workout query, delete-nav target fixes, dead "group by program" template-list code removed. Found as uncommitted local work at session start, unknown prior authorship; verified via Playwright (48/48) + 3-agent review before push. **Pushed 2026-07-02 (0a0f89f).** |
| app-programs v=4 / calendar-goals v=2 / workouts v=5 / runner v=3 | Periodization (Linear/Undulating) + assignment-time 1RM check + solo RLS fix + inline assign-workout grid (replaces the old modal) — see STATUS "What's working" for full detail. **Pushed 2026-07-01 (76cb53f).** |
| app-workouts v=3 | Runner template list limit raised to 2000; startWorkoutRunner fetches template by ID to bypass max_rows=200 cap |
| modular | app.js (7,968 lines) split into 8 modules; pre-push hook updated; preview server path fixed; .gitignore updated |
| v180 | iOS Safari session detail slide-in fix — `inset:0` → explicit `top/right/bottom/left` |
| v179 | `sudoAsClient()` in-function email guard (security fix); session detail slide-in smoke test |
| v178 | Re-fix session detail slide-in — panel wrapper changed to `position:fixed;inset:0;z-index:1000` |
| v177 | Solo 1RM library fix (was gated by isClientPlan); propagation toast for shared templates |
| v176 | `openSessionDetail()` slide-in drawer; 39/39 Playwright green; 8 solo smoke tests |
| v175 | PT dashboard activity feed scoped to coach's clients only — solo sessions no longer show as "Unknown" |
| v174 | Sudo/impersonation mode — "View as" button on client list, amber banner, exitSudo() |
| v173 | Dashboard layout rework (all 3 dashboards); hero card, stats strip, two-column grid; Progress tab add buttons |
| v162 | Solo Workouts shows program session accordion (renderClientWorkoutsPage); renderWorkoutTemplates excludes client_id-tagged templates |
| v161 | Start button on template detail for solo + client context; sql-safety RLS role audit section; hello-claude golden path walk behaviour |
| v160 | Solo Programs audit — "Add to my plan" button; showAssignProgramToClientModal solo path; empty state copy |
| v159 | Fix calendar ReferenceError (currentView not defined — broke all roles) |
| v158 | Personal account (solo view) — three-pill view switcher, solo dashboard, _getCurrentClientId helper, progress functions fixed, privacy policy page |
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
