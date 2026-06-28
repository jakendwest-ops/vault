# CoachApp — ACTIVE PROJECT

> PTHub is **frozen** — reference spec only. No new features built there.
> All active development is in CoachApp.
> Full product blueprint: `Vault/projects/CoachApp/blueprint.md`

**Location:** `C:\Users\jaken\coachapp\`
**Git:** master branch — pushed to https://github.com/jakendwest-ops/coachapp
**Stack:** Vanilla JS + Supabase (project: `avilxuiacmtgeoxxhfhc`)
**Preview:** `localhost:3001` — via `preview_start("CoachApp")`
**Live:** https://jakendwest-ops.github.io/coachapp (GitHub Pages, auto-deploys on master push)
**Auth:** Email/password, email confirmation OFF
**Deploy:** GitHub Pages — push to master, auto-deploys
**Cache-busting:** Bump `?v=N` on app.js script tag in index.html on each commit (currently **v=143**)

---

## Supabase schema (confirmed current)

`profiles` (role: coach | solo | client), `clients` (coach_id, user_id nullable FK to profiles.id, invited_at timestamptz),
`weight_logs`, `goals` (created_by NOT NULL FK → profiles), `goal_milestones`, `goal_check_ins` (created_by FK → profiles),
`exercises` (+ muscle_groups text[]), `sessions`, `session_exercises`,
`workout_logs` (+ `notes` text for coach session notes), `workout_log_exercises` (+ `client_notes` text — migration run 2026-06-28), `workout_log_sets`,
`workout_templates`, `workout_template_exercises` (sets_json jsonb, + one_rm_kg decimal, + `superset_group` text, + `bodyweight` bool, + `assisted` bool, + `notes` text),
`client_programs` (id, client_id, program_id, start_date, status, created_at),
`client_1rms` (id, client_id uuid FK→clients.id, exercise_name text, one_rm_kg decimal, recorded_at timestamptz default now()),
`events` (is_pt_assigned boolean, type: session|review|competition|holiday|gym|other, created_by FK → profiles, client_id nullable),
`performance_logs` (category: strength|cardio|body_metric|benchmark, value, unit, logged_by FK → profiles),
`programs` (coach_id), `program_phases` (program_id, name, order_index),
`program_phase_workouts` (phase_id, template_id, day_of_week int, day_label text, notes text, session_order int default 1),
`client_check_ins` (client_id, sleep, energy, stress, soreness 1–5, notes, created_at),
Supabase Storage bucket: `progress-photos`

**Cardio set fields in sets_json (workout_template_exercises):**
`pace500Min`, `pace500Max`, `restMin`, `restMax`, `hrZoneMin`, `hrZoneMax`, `duration`, `distance`, `isDistanceBased`
`paceKmMin`, `paceKmMax` — running pace per km (MM:SS string)
`restHrMax` — max heart rate before starting next interval (int, BPM)
`strokeRateMin`, `strokeRateMax` — rowing/ski stroke rate (int, spm)
`exercise_type` column: `'cardio'` or `'strength'`

**`workout_log_sets.set_type` check constraint:** Column has a DB-level check constraint `workout_log_sets_set_type_check`. The value `'working'` violates it and causes a silent 400 error that drops all set data while the parent session row still saves. **Do not insert `set_type` — omit it entirely and let the DB default.**

**FK constraint (confirmed present):** `workout_log_sets_exercise_id_fkey` already exists — confirmed by attempting to add it and receiving "constraint already exists" error. The two-query workaround in `openWorkoutLog` is now redundant and can be simplified to a nested join in a future cleanup session.

**Profiles FK map (all 8 columns):**
clients(coach_id), clients(user_id), goals(created_by), goal_check_ins(created_by), sessions(coach_id), events(created_by), performance_logs(logged_by), programs(coach_id)

**Supabase functions (confirmed clean):**
- `handle_new_user()` — trigger on auth.users INSERT; `security definer set search_path = ''`; creates profiles row only; `on conflict do nothing`
- Edge Function `invite-client` — stamps `user_id` + creates profile + sets `invited_at` at invite-send time using service role. **Note:** times out when called from localhost preview (30s); use live site for invite flow.
- `log_audit_event()` — trigger on 17 tables; logs all INSERT/UPDATE/DELETE to audit_log with old_data/new_data jsonb

---

## RLS policies (confirmed current — optimised 2026-06-23)

All policies now use `(SELECT auth.uid())` wrapper for ~95% per-row speedup.
10 duplicate SELECT policies dropped.
New `Client inserts own workout log sets` policy added (was missing — client self-logging would have silently lost set data).

**Coach policies** — ALL operations scoped to `coach_id = (SELECT auth.uid())` or equivalent
**Client policies** — SELECT + INSERT/UPDATE policies using `IN (SELECT id FROM clients WHERE user_id = (SELECT auth.uid()))` anchor pattern
**Key rule:** Never reference `auth.users` table directly in policies — use `auth.uid()` and `auth.email()` (JWT functions, no table access needed)
**Storage:** `progress-photos` bucket has INSERT (client upload) and SELECT (coach + client) policies via SQL Editor

---

## Code quality infrastructure

**Pre-push git hook** — delegates to `scripts/checks.sh` (tracked file). Catches:
1. JS syntax check (`node --check`) — blocks push if app.js won't parse
2. Wrong column names (logged_at on weight_logs/workout_logs, coach_notes)
3. Unscoped multi-tenant queries (clients, workout_templates, programs)
4. Missing cache-bust version in index.html
5. Bare `alert()` calls — use `showToast()` instead
6. Hardcoded UUIDs or emails
7. `set_type:` in inserts — DB constraint rejects it
8. `if (setsErr) log.error` with no abort — swallowed write error
9. Bare `clearInterval()` — always use `x = clearTimer(x)` to null the variable
10. Duplicate function definitions

**GitHub Actions CI** (`.github/workflows/deploy.yml`) — runs `scripts/checks.sh` on every PR and push. Deploy job only fires after checks pass.

**`clearTimer` helper** — `const clearTimer = id => { clearInterval(id); return null }` — defined near top of app.js. Always use this instead of bare `clearInterval()`.

**`dbq()` wrapper** — auto-logs all Supabase errors with timing; demotes PGRST116 to warn; shows user toast on error. All write paths confirmed with error handling — sweep complete.

**`setsHadError` flag** — tracks set insert failures in `saveRunnerSession` so success toast only fires if all sets saved cleanly.

**`_activePage` localStorage** — persists current page across refresh for all roles.

---

## E2E test infrastructure

**Playwright** — 14 tests, all passing clean (no retries). Run: `npm test` in `C:\Users\jaken\coachapp`.

**Test accounts:**
- PT: `coachapp.e2e.pt@gmail.com` / `E2eTestPass123!`
- Client: `coachapp.e2e.client@gmail.com` / `E2eTestPass123!`

**Seed script:** `node scripts/seed-test-data.js` — idempotent, creates accounts + dummy data.

**Config:** `playwright.config.js` — viewport 390×844, retries 1, screenshot + video + trace on failure.

**Key files:**
- `tests/fixtures.js` — shared fixture; ALL specs import `{ test, expect }` from here (auto console error capture)
- `tests/helpers.js` — `loginAsPT` / `loginAsClient`
- `tests/auth.spec.js`, `tests/pt-dashboard.spec.js`, `tests/client-workout.spec.js`

**Dummy E2E client (PT-side manual testing):**
- Name: "Test Client", client ID `8442071c-03e1-40e2-b88d-9a2730aa1de4`
- Email: `jakendwest+testclient@gmail.com`, Hyrox Experiment assigned from 2026-06-29
- No auth login — PT-side testing only. For client-side E2E use Jake's master account in client view.
- Invite timed out on localhost — retry from live site when needed.

**Smoke test result (2026-06-28 v143):** All 9 checks PASS — PT programs accordion, client calendar, day modal, runner, set logging, finish screen, save navigation.

**Future Playwright scenarios to add:**
- **Program reassignment clone safety** — assign → edit client sessions → unassign → reassign → verify edits gone and templates deleted
- **Edit start date** — assign with date A → edit to date B → verify calendar shifts correctly

---

## DB state (current)

- `clients`: Alex Turner (jakendwest+test@gmail.com), Sarah Mitchell, Jake West (master account), Test Client (jakendwest+testclient@gmail.com — dummy E2E profile)
- Jake's client record id: `97bb871a-9c2b-44d1-aad8-c188c2729105`
- Jake's user id (coach_id): `c930ce7f-3ffd-4b1e-9d7b-2bcb226f4954`
- E2E PT user id: `faab90b2-d277-4feb-b0d6-c2cecaeab76d`
- E2E Client user id: `2db5dbb6-af69-4181-9650-12576ed112f6`
- **Hyrox Experiment program** (program_id: `2db32003-c020-4f8b-8c0a-d6dfde7c203e`) — assigned to Jake West (start 2026-06-29) and Test Client (start 2026-06-29)
- **`client_1rms` table** — Jake's 1RMs: Bench 115kg, Squat 160kg, Deadlift 210kg, Military Press 87.5kg
- **`session_order` column** added to `program_phase_workouts` (int, default 1). AM=1, PM=2.

---

## What CoachApp has (confirmed working)

- Auth — login / signup / session persistence
- Client list + client profile (Overview / Goals / Workouts / Weight / Performance / Programs / Photos tabs)
- Goals + milestones (interactive toggle) + check-ins + inline goal progress Update button
- Exercise library (global per-coach)
- Workout templates — create / edit / delete
- Exercise-in-template set form (AMRAP/Uni/Timed, Reps, Weight, %1RM, Rest MM:SS, RPE/RIR, Tempo, Countdown, Notes)
- **Template set editor** — "Copy last set ↑" button on set 2+, BW/Assisted toggles, superset_group input
- **Cardio set form** — Pace/500m, Pace/km, HR Zone, Rest, Rest HR max, Stroke rate (spm), Duration, Distance
- **Template card set preview** — shows full detail per set
- **Section labels** — `[WARM-UP]` / `[MAIN SET]` / `[COOL-DOWN]` prefix in notes field
- Log workout against template (retrospective modal)
- **Workout Runner** — real-time gym logger with cardio mode, interval timer, rest timer, bodyweight/assisted/superset support
- **Runner target chips** — duration (normalized MM:SS from raw seconds), pace/500m, pace/km, rest, stroke rate, rest HR max
- **Session history view** — two-query fetch + manual merge
- **Session summary / finish screen** — PR badge detection, stats row, exercise cards
- **PT dashboard** — compliance card, activity feed, filter tabs, recency labels
- **Client dashboard** — this-week banner, check-in card, recent sessions, weight chart, weekly check-in
- **Client calendar** — full monthly grid; program workouts mapped to dates; day detail modal with SESSION 1/2 labels, exercise list always visible, Start buttons (v143 — smoke tested)
- **Client Workouts page** — Phase → Day groups (sorted Day 1–6) → SESSION N/M labels → exercise list + Start button (consistent with PT view)
- **PT Client Programs accordion** — Phase → Day groups → SESSION N/M → exercise list (name + set count) + Edit button per session
- **Edit start date** — PT can change a program's start date from client Programs tab; shifts entire calendar instantly
- **Remove program** — PT can unassign program from client (clears calendar)
- **Client day detail modal** — SESSION 1/2 labels, exercise list always visible, Start workout button (consistent with PT accordion style)
- **Client plan editing** — PT can edit client's program sessions; back nav preserved; "apply to all same-name sessions" propagation
- **Timed sets** — template builder timed toggle; runner shows duration field for timed exercises
- **Unilateral L/R logging** — runner shows separate L/R weight+reps columns; logged sets display L/R format
- **Runner UI redesign** — "Set X of Y" in header; visible Edit button on logged sets; PT note always shown; client notes textarea per-exercise
- **Playwright runner suite** — PT workouts, client runner, finish screen, save navigation, skip rest, accordion coverage
- Programs — phase-based builder, assign to client with start date
- Calendar — coach creates events; client calendar tab added
- Settings page — profile name edit, change password
- Progress photos, weight tracking, performance/PB tracking
- Invite system — fully working (live site only; times out on localhost)
- Structured console logging + DB-side audit log
- Pre-push git hook + GitHub Actions CI
- Master account view switcher (PT|Client)

---

## What to build next

### Near-term (Week 3 — Jul 8–14)
- Branding — logo upload, display on dashboards
- Goals overhaul — granular mini-goals and milestones
- UI consistency pass
- Metric/imperial toggle
- 💡 Future — Individual session skip/move on client calendar (defer until real PT usage data)

### Beta prep (Week 5 — Jul 22–31)
- Full walkthrough, Playwright suite, Supabase redirect URL audit
- Beta invites staggered: Jul 25, Jul 28, Jul 31

---

## Open to-dos for Jake

- **Daily question cron** — fires at 9:07am while Claude Code is open; session-only (recreate if app restarts)
- Check Supabase dashboard → Logs → API after each test session for PGRST errors
- **Verify client calendar on live site** — smoke tested in preview v143; confirm once on live
- **Client auth for Test Client** — invite to jakendwest+testclient@gmail.com timed out on localhost; retry from live site to complete the client-login E2E path

---

## Session protocols

### HELLO CLAUDE (session start)
Skill: `C:\Users\jaken\coachapp\.claude\skills\hello-claude\SKILL.md`

Steps (run in order, none skipped):
1. `preview_start("CoachApp")` + resize to 480×844; check daily question cron active
2. Read Vault: STATUS.md + LOG.md + roadmap.md + blueprint.md + **lessons.jsonl + voice.md** (all mandatory)
3. Summarise last session + CI deploy status
4. **Automated code review** — grep app.js for wrong columns, unscoped queries, client coach_id errors, role routing gaps, duplicate functions, hardcoded IDs
5. Roadmap cross-check — flag items done in LOG but still marked planned in roadmap
6. Propose plan for this session
7. **Verify** open to-dos against current evidence before surfacing — close resolved items explicitly
8. **Predictions review** — check predictions.jsonl for expired verify_by dates

### WORKING AGREEMENT
Before building anything: discuss, agree on approach, then present **one full plan summary** for Jake's explicit approval. Do not start building until Jake says "approved" or equivalent.

### /SAVE (session end)
Skill: `C:\Users\jaken\coachapp\.claude\skills\save\SKILL.md`

### /PLAYWRIGHT (run E2E tests)
Skill: `C:\Users\jaken\coachapp\.claude\skills\playwright\SKILL.md`

### /DEPLOY-CHECK (before beta invites)
Skill: `C:\Users\jaken\coachapp\.claude\skills\deploy-check\SKILL.md`

### SQL SAFETY (before any SQL)
Skill: `C:\Users\jaken\coachapp\.claude\skills\sql-safety\SKILL.md`

### MOBILE CHECK (after any UI change)
Skill: `C:\Users\jaken\coachapp\.claude\skills\mobile-check\SKILL.md`

---

## Correct technical reference (verified)

- `inviteUserByEmail()` returns `data.user.id` immediately — stamp in Edge Function at send time
- Supabase JS v2 auto-processes invite hash on page load — no manual `setSession` needed
- Trigger pattern: `security definer set search_path = ''`
- RLS policies: use `(SELECT auth.uid())` — wrapping in a subquery caches the JWT call once per query vs once per row
- RLS subqueries: always use `IN`, never `=`, for subqueries that could return multiple rows
- GitHub Pages: push to master, auto-deploys. No config needed.
- DO blocks in SQL editor roll back entirely on any error — use plain statements for destructive work
- `client_programs` columns: id, client_id, program_id, start_date, status, created_at
- coach_id for client self-logging: derive from `clients.coach_id` — do NOT use `currentUser.id`
- `workout_logs` columns: `date` (not `logged_at`), `created_at`, `notes` (not `coach_notes`)
- `weight_logs` date column: `created_at` (not `logged_at`)
- Client-side workout_templates queries: must use `clients.coach_id`, not `currentUser.id`
- Storage policies must be added via SQL Editor, not the Supabase Storage UI dialog
- Preview viewport: always resize to 480×844
- `onAuthStateChange` fires `SIGNED_IN` on every token refresh — guard with `_appLoaded` flag
- `saveRunnerSession` after save: check `currentView === 'client'` → `navigate('workouts')`; else `openClient(clientId)`
- Playwright: `#workout-runner` has zero height — assert on child elements
- Section labels: `[WARM-UP]` / `[MAIN SET]` / `[COOL-DOWN]` prefix convention in `notes` field
- `window._phaseWorkoutContext` — carries `{ phaseId, dayOfWeek }` from phase-assign modal into template create flow
- `_runner.templateDesc` — carries template description into runner for session brief display
- **Client calendar:** `window._calProgramWorkouts` stores programWorkoutsByDate; `window._calClientId` stores clients.id — both set in `renderCalendar` client branch, read by `showClientDayDetail`
- **Template filter:** PT workouts list filters to `!t.program_id` only — week-specific templates hidden from flat list
- **`normalizeDuration(v)`** — converts raw seconds string ("1800") or number to MM:SS; use for any cardio duration display
- **Day grouping pattern:** sort `program_phase_workouts` by `day_of_week` → `session_order`; group into `dayMap`; render DAY N rows with SESSION N/M labels when `daySessions.length > 1`
- **`isClient` check in calendar:** use `currentProfile?.role === 'client' || currentView === 'client'` — master account has role 'coach' even in client view
- **`clients.id` vs `currentUser.id`:** `client_programs.client_id` = `clients.id` (different UUID from auth uid) — always fetch `clients.id` first

---

## Bug patterns to watch

- `weight_logs` queried with `logged_at` — use `created_at`
- `workout_logs` queried with `logged_at` — use `date` or `created_at`
- `log.coach_notes` — column is `notes`
- Client user querying coach-owned tables with `currentUser.id` as coach_id — derive from `clients.coach_id`
- Unscoped queries on `workout_templates`, `clients`, `programs` — must include `.eq('coach_id', ...)`
- Null role in profiles (invited clients) — always handle gracefully in role routing
- Bare `await db.from(...)` with no `{ error }` destructure — silent failure, use `dbq()`
- `set_type:` in any `workout_log_sets` insert — DB check constraint rejects it; omit the field entirely
- `if (setsErr) log.error(...)` with no `return`/`throw`/flag — always abort or set `setsHadError = true`
- `clearInterval(x)` without `x = null` — always use `x = clearTimer(x)`
- `auth.uid()` bare in RLS policies — wrap as `(SELECT auth.uid())`
- Post-save navigation: save functions must not call `openClient()` directly — check `currentView`
- PT/client render parity: when updating an interactive list in PT view, grep for the client-view equivalent
- Cardio duration stored as raw seconds string — use `normalizeDuration()` before display, not raw value
- Modal inner div: use class `modal` (not `modal-box` — no CSS for that); outer div uses `modal-overlay`

---

## Slash commands

| Command | When |
|---|---|
| `hello claude` | Start of every session |
| `/save` | End of every session |
| `/code-review` | Before every git push (offer) + mandatory in /deploy-check |
| `/playwright` | After any significant change; before deploy |
| `/deploy-check` | Before any beta invite or significant public push |
| `/sql-safety` | Before writing any SQL |
| `/mobile-check` | After any UI change, before reporting done |

---

_Last updated: 2026-06-28_
