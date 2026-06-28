# CoachApp / PTHub — LOG

## 2026-06-28 — UI consistency pass, E2E smoke test (v135–v143)

**Done:**
- **v136-v137 — Client day detail modal fix:** Modal was using `class="modal-box"` (no CSS) → fixed to `class="modal"`. Day detail now readable and styled.
- **v138 — Collapsible exercise list in day detail modal:** Added per-session exercise list (name + set count, always visible, no chevron). Jake chose option A: name + count only, not full set details.
- **v139 — Edit start date:** PT can change a client's program start date from the Programs tab. `showEditStartDateModal` / `saveEditStartDate` added. Helper text explains calendar shift. Verified in preview.
- **v140 — PT Client Programs — day grouping + SESSION labels:** Phase rows now group by `day_of_week`, sorted Mon–Sat; sessions on the same day get SESSION 1/2 labels. Exercise list (name + set count) always visible per session. Edit button per session.
- **v141 — Client day modal — consistency with PT view:** SESSION 1/2 labels, exercise list always visible, Start button. Removed per-session chevron toggle. Now matches PT accordion style exactly.
- **v142 — Client Workouts page — consistent with PT accordion:** Phase → Day groups (sorted by day_of_week) → SESSION N/M labels → exercise list → Start button. Same pattern across all three surfaces (PT Programs, client calendar modal, client Workouts).
- **v143 — Cardio duration normalization:** `normalizeDuration(v)` added — converts raw seconds ("1800") or number to MM:SS. Applied to runner target chip and cardio duration input value/placeholder. Fixes "1800" displaying instead of "30:00".
- **`client_notes` SQL migration confirmed:** Jake ran the migration successfully ("SQL success" confirmed this session).
- **Test Client created:** Dummy client "Test Client" (client ID `8442071c`) added to Jake's PT account. Hyrox Experiment assigned from 2026-06-29. Client fully cloned via `_cloneProgramForClient`.
- **E2E smoke test — all 9 checks PASS:**
  - PT → Test Client overview (Hyrox active) ✅
  - PT → Programs tab (12 phases, day groups, SESSION 1/2) ✅
  - Client calendar (June 29, AM/PM workouts) ✅
  - Client day modal (SESSION 1/2, exercise list, Start buttons) ✅
  - Workout runner (7 exercises, set targets, coach note, 1RM chip) ✅
  - Set logging (75kg × 5 reps, rest timer, volume 375kg) ✅
  - Finish screen (PR badge, summary, Save button) ✅
  - Post-save navigation → Workouts page ✅

**Bugs found + fixed:**
- `showClientDayDetail` used `class="modal-box"` with no CSS definition — modal was unstyled
- PT Programs sessions rendering out of order — was sorted only by `day_of_week`, not also `session_order`. Fixed: sort by `[day_of_week, session_order]` before grouping.
- Cardio duration target chip showing raw "1800" seconds — added `normalizeDuration()` to handle both string seconds and already-formatted MM:SS values

**Decided:**
- Consistency pattern agreed: all three surfaces (PT Programs, client day modal, client Workouts) use identical structure: Day group → SESSION N/M label (when multi-session) → session name → exercise list → action button. Why: Jake explicitly said he preferred the PT UI and wanted client UI to match.
- SESSION N/M labels only shown when `daySessions.length > 1` — single-session days don't show the label. Why: avoids visual noise on days with one workout.
- Test Client invite flow deferred: Edge Function times out from localhost (30s). Retry from live site. Client-side E2E uses Jake's master account in client view as workaround.

**UNVERIFIED (banked):**
- Client calendar on live site — smoke tested in preview v143; Jake confirmed intent to verify on live. Still pending.
- Test Client auth — invite to jakendwest+testclient@gmail.com timed out on localhost; client-login E2E path not tested.

## 2026-06-27 — Runner redesign, client plan editing, timed/unilateral sets, accordion UI (v122–v134)

**Done:**
- **v122 — Bug fixes:** PT workouts page blank-page guard when all templates have program_id; runner `targetSets` fix (`ex.sets_json?.length` not `ex.sets`); iOS beep keep-alive (`_unlockAudio()` on every rest tick); `performance_logs` `.order('date')` fix (was `logged_at` — silently returning no PBs)
- **v123 — Cardio/strength display fixes:** `parseRest()` for cardio duration (was showing 5:00 as 0:05); template drawer strength set fields fixed; rest timer readability (14px font, shows Xs below 60s)
- **Playwright runner test suite added** — PT workouts, client runner, finish screen, save navigation tests
- **v129 — Client plan editing:** Back button returns to correct page; Client Overview shows active program name; Client Programs tab redesigned with full phase structure + per-session Edit buttons; "Editing [name]'s plan" banner; on save, prompts "apply to all same-name sessions in this client's plan"; no auto-renaming
- **v130 — Programs tab accordion:** Client Programs tab shows phases as collapsible panels; Edit exercise modal shows client's 1RM exercises as dropdown (select → auto-fills name field)
- **v131 — Timed sets + unilateral L/R logging:** Template builder timed toggle swaps Reps for Duration (mm:ss) field; runner input handles timed (duration field) and unilateral (separate L/R weight+reps columns); logged sets display duration (⏱) and L/R format; client workouts page replaced flat list with accordion phase panels + Start button per session (falls back to flat if no program)
- **v132 — Runner UI redesign:** Header shows "Set X of Y" inline; visible "✎ Edit" button on logged sets; PT note always shown in scrollable area (section label prefix stripped); "Your notes" textarea per-exercise, persists across set logs, saves to `workout_log_exercises.client_notes`
- **v133 — Skip rest fix:** `skipRestTimer()` now calls `renderRunner()` when no `_afterRest` callback set — rest block was not clearing on skip
- **v134 — Client accordion regression fix:** Session rows now show exercise count + day label; tapping name expands to show exercise list with set summaries; `workout_template_exercises` fetched in initial query
- **Playwright tests strengthened:** Skip rest test asserts rest overlay gone + LOG button visible; accordion tests cover phase expand + session exercise detail; stale selector fix

**Bugs found + fixed:**
- PT workouts blank page when Hyrox program covers all templates — added standalone-empty guard
- Runner set counter didn't advance past set count — `ex.sets` doesn't exist on `workout_template_exercises`; fixed to `ex.sets_json?.length`
- iOS AudioContext auto-suspend killing beeps on exercise 2+ — keep-alive call on every rest tick
- `performance_logs` PBs tab returning empty — `.order('logged_at')` → `.order('date')`
- Cardio duration showing 0:05 instead of 5:00 — `parseInt()` → `parseRest()`
- Rest overlay not clearing on skip — `skipRestTimer()` missing `renderRunner()` call
- Client accordion not expanding session detail — `workout_template_exercises` not in query

**Decided:**
- No auto-renaming of templates when editing a client plan — names only change when PT explicitly edits them. Why: Jake confirmed this was intentional architecture. See [[project-coachapp-architecture]].
- `workout_log_exercises.client_notes` column added — needed for runner per-exercise notes save. **SQL migration required before this works on live.**

**UNVERIFIED (banked):**
- `client_notes` column — `saveRunnerSession` inserts it but the column may not exist in production yet. Supabase will return a 42703 error if missing. Jake needs to run: `ALTER TABLE workout_log_exercises ADD COLUMN IF NOT EXISTS client_notes text;`
- Client accordion expand in Playwright — tests skip gracefully if E2E client has no program assigned; actual live coverage pending program assignment to test account.

## 2026-06-26 — 12-week program fully built, client calendar, template filter (v120–v121)

**Done:**
- **12-week Hyrox Experiment — all phases built (v120)** — Deleted 44 incorrect Option A templates (plan's exercises) and rebuilt with Option B: Jake's preferred DB exercises (Bench Press, Dead-Stop Barbell Row, Back Squat, Conventional Deadlift, Military Press, DB Bulgarian Split Squat, Barbell Hip Thrust, etc.) with plan's progressive weights on main lifts per week. Phases 2–12 created. Cardio templates (Row/Run/SkiErg × Threshold/Aerobic) reused across all phases. All 11 weeks returned `{ok: true}`.
- **`client_1rms` table created** — Jake's 1RMs entered: Bench 115kg, Squat 160kg, Deadlift 210kg, Military Press 87.5kg.
- **Client calendar tab added to nav (v121)** — Calendar was in `clientPages` array but missing from `_NAV_ITEMS.client`. Added between Workouts and Progress.
- **Client day detail modal** — tapping a calendar day opens a session detail modal with AM/PM labels and a `▶ Start workout` button per session. Rest days show a rest message. `showDayEvents` now routes to `showClientDayDetail` for clients instead of the add-event modal.
- **Template ID added to client calendar query** — `workout_templates(name)` → `workout_templates(id, name)` so Start button has the ID.
- **`window._calProgramWorkouts` + `window._calClientId`** — stored in `renderCalendar` client branch so `showClientDayDetail` can read workout data and pass correct client ID to runner.
- **Dashboard program strip removed** — `renderClientProgramCard` and `clientProgramNav` functions removed. Client dashboard no longer shows the navigable week card; Calendar tab is the replacement.
- **PT workouts template filter** — program-linked templates (any with `program_id` set) hidden from flat list. Only standalone templates appear. Week-specific templates accessible via Programs → phase view only.
- **Run Threshold distance** — verified already `distance: "1"` in DB. No fix needed.
- **Progression rules** added to roadmap as `💡 Future` item.

**Decided:**
- Option B for 12-week program: use Jake's preferred exercises from existing Phase 1 DB templates, apply plan's progressive weights to main lifts. Why: Jake had chosen those exercises deliberately.
- Progression rules engine deferred to future. Why: current 44-template-per-week approach works; a rules engine is complex and no immediate PT demand. Better path is a faster template builder UI.
- Template filter Option A (hide program-linked templates from flat list entirely). Why: simpler, cleaner — week templates belong in program context, not a flat browse list.
- **New working agreement:** discuss + approve before building. Jake now asks for a full plan summary and gives explicit approval before any build starts. Prevents rework from incomplete information.

**UNVERIFIED (banked):**
- Client calendar tab, day detail modal, Start button flow — pushed as v121 but not visually confirmed in browser. Jake testing on live site.
- PT workouts template filter — pushed v121, not visually confirmed.

## 2026-06-25 — Hyrox Experiment in DB, AM/PM sessions, runner limitations fixed (v99–v116)

**Done:**
- **AM/PM session support** — `session_order` column added to `program_phase_workouts` (int, default 1; AM=1, PM=2). Schedule, calendar, and phase builder all use it as source of truth.
- **Hyrox Experiment program built in DB** — Upper A (horizontal push/pull), Lower A (squat), Upper B (vertical push/pull + stations), Lower B (deadlift pattern) strength templates created. 9 sessions assigned Mon–Sat with correct AM/PM order. Existing cardio templates (Row/Run/SkiErg × Threshold/Aerobic) wired in.
- **Runner: distance input (metres)** — exercises matching `/carry|broad jump|sled|sandbag.*lunge/i` now show a Metres input instead of Reps. Logged as `distance_m`. Target extracted from coach notes (e.g. "30–50m per set").
- **Runner: PM ▶ Start button** — all sessions on today get a Start button (was only AM/first session).
- **Calendar: program workouts** — client calendar now maps phase workouts to actual calendar dates using program start_date. Shows workout name (truncated after " —") with AM/PM labels for double-session days.
- **Session slot validation** — saving a phase workout to a slot already taken shows a toast and blocks insert. Prevents duplicate AM/AM or PM/PM on same day.
- **deleteTemplate root-cause fixed** — `renderWorkoutTemplates` was missing `.eq('coach_id', currentUser.id)` filter; foreign templates leaked into the list causing deletes to fail silently.
- **JS syntax check added to pre-push script** — `node --check js/app.js` now runs as check #1; catches syntax errors before they reach production.
- **Mobile breakpoint 768px → 900px** — fixes preview always showing desktop sidebar layout.
- **`fix_session_order.cjs`** — one-time script to dedupe accidental re-run of setup script and set correct `session_order` values.

**Bugs found + fixed:**
- `const sessionOrder` declared twice in `savePhaseWorkout` → `SyntaxError` → prod wouldn't load. Fixed by removing duplicate declaration. JS syntax check now prevents this class of bug.
- Accidental re-run of `run_setup.cjs` created duplicate Upper A and Lower A templates + duplicate Hyrox Experiment program. Cleaned via `fix_session_order.cjs`.
- Saturday session_order was backwards (SkiErg=1, Lower B=2). Fixed to Lower B=1 (AM), SkiErg=2 (PM).
- Service role key `sb_secret_kVcuv…` was committed to scripts and exposed in chat. Key revoked. Scripts now use `process.env.SUPABASE_SERVICE_KEY`. New key: `sb_secret_65IsZM…`.
- GitHub push protection blocked push due to key in git history. Resolved by soft-resetting all local commits to `origin/master` and recommitting cleanly.

**UNVERIFIED (banked):**
- Calendar workout display: code written and pushed but not visually confirmed in browser — Jake's client view may need to be checked to confirm workouts appear on correct dates.
- Distance runner input: logic added for Farmer's Carry / Burpee Broad Jump but not tested in a live runner session.

**Decided:**
- 12-week Hyrox program: 12 phases (one per week), 1RM-based weight calculation. `client_1rms` table + runner intensity → weight calc. Next session dedicated build.
- Service role key stays out of all committed code — always use `process.env.SUPABASE_SERVICE_KEY`.

**Why:**
- 1RM-based calc means the program works for any athlete, not just Jake. Jake confirmed he wants everything in platform, no PDFs.

## 2026-06-25 — Hyrox Week 1 build, cardio template overhaul, expandable client cards (v97–v98)

**Done:**
- **"+ Create new workout" button in phase-assign modal (v96)** — opens template builder directly; `window._phaseWorkoutContext` carries `{ phaseId, dayOfWeek }` into the create flow; `saveNewTemplate` branches on context and calls `openTemplate(id)` instead of reopening the assign modal
- **Hyrox Test program — Week 1 built** — 6 templates created (Row Threshold, Run Aerobic, SkiErg Aerobic, Run Threshold, Row Aerobic, SkiErg Threshold) and assigned Mon–Sat in Base Building phase. Each has warm-up / main set / cool-down structure with section labels, coaching notes, and set targets.
- **Fix all 8 cardio template gaps (v97):**
  - Duration displayed as MM:SS (was raw seconds e.g. "600")
  - Rest displayed as M:SS (was raw seconds e.g. "90")
  - Section labels parsed from `[WARM-UP]` / `[MAIN SET]` / `[COOL-DOWN]` prefix in notes — rendered as accent badge chips in template cards
  - Pace/km field added to cardio set form (`paceKmMin` / `paceKmMax`)
  - Rest HR max field added (`restHrMax`, BPM)
  - Stroke rate field added (`strokeRateMin` / `strokeRateMax`, spm)
  - Runner target chips extended: paceKm, strokeRate, restHr chips added; duration + rest display fixed to use `fmtDuration()` when stored as integer
  - Template description shown as session brief in runner header (`_runner.templateDesc`)
- **Client workout cards — expandable detail (v98)** — tap card to expand/collapse; shows section label badges, exercise name, set targets (duration/pace/HR/stroke rate), coaching notes; chevron indicator rotates; Start button has `event.stopPropagation()` so it doesn't toggle

**Bugs fixed during build:**
- `exercise_type` column queried on `exercises` table — doesn't exist there, only on `workout_template_exercises`; removed from query
- `null value in column "exercise_name"` on insert — column is NOT NULL; fixed by adding `exercise_name: ex.exKey` to insert
- Stale phase ID in phase-assign modal — DOM modal persisted from before page reload, showing old UUID; resolved by querying `program_phases` fresh
- `saveNewTemplate` phase-context branch was re-opening assign modal instead of going to template editor — user hit "this button doesn't take me to the create new template section"; fixed to call `openTemplate(data.id)`

**UNVERIFIED:**
- Run Threshold template — 4 main set rows stored with `distance: 1000` (treated as meters); app expects km. Should be `distance: 1`. Fix: edit rows directly in Supabase table editor. To-do banked in STATUS.

**Decided:** Section label convention (`[WARM-UP]` prefix in notes) is the standard for all cardio template exercises going forward.
**Why:** Avoids a new DB column; section context travels with the exercise note; works for both template card display and client expandable view with no extra queries.
**Revisit-if:** Jake wants per-exercise section field in DB (unlikely — he approved the notes prefix approach without hesitation).

## 2026-06-24 — hello-claude + /save skill audit and full overhaul (no version bump)

**Done:**
- **Root cause of stale to-dos diagnosed and fixed** — two failure modes: (1) /save ritual wasn't running regularly; (2) hello-claude Step 7 listed to-dos without verifying them. Both fixed.
- **hello-claude Step 7 rewritten** — now verifies each to-do against current evidence (CI run list, Playwright coverage, Supabase schema) before surfacing it. Closes resolved items explicitly.
- **hello-claude Step 8 added** — predictions review: scans predictions.jsonl for expired verify_by dates, surfaces for grading.
- **hello-claude Step 5 added** — roadmap cross-check: flags items that are done in LOG but still marked planned in the roadmap.
- **hello-claude Step 2 fixed** — lessons.jsonl and voice.md now mandatory reads (were "if needed").
- **hello-claude standing behaviours updated** — `programs` table added to unscoped query scan; "pending push" wording replaced with CI check; `educate` skill linked explicitly; all-caps = patience exhausted signal added; daily question cron mid-session handling documented.
- **Global hello-claude fully synced** — was 6 steps with outdated standing behaviours; now identical to CoachApp version (8 steps, all behaviours, "Who Jake is" section).
- **/save Step 3 rewritten** — now sweeps existing to-dos for resolved items before adding new ones; states what was cleared and why.
- **/save Step 6 rewritten** — only lists genuinely open items; explicitly states what evidence would close each; empty list is called out as a good outcome.
- **/save** — UNVERIFIED→to-do cross-link added (Step 4 now tells the LOG to create a corresponding to-do for any UNVERIFIED item banked).
- **STATUS.md session protocols** — updated to reflect 8-step hello-claude with all new steps.

**Decided:** Every to-do is a claim to be verified, not a fact to be repeated. Both hello-claude and /save now treat surfacing a to-do as a verification action, not a copy action.
**Why:** Two stale items persisted for days undetected because nothing in the workflow ever checked whether they were still true. The fix is structural — verification is now the default, not an optional extra.
**Revisit-if:** A new class of to-do item emerges that can't be auto-verified — add it to the verification ladder in hello-claude Step 7.

## 2026-06-24 — Playwright E2E suite, test infrastructure, skills overhaul (v86–v93)

**Done:**
- **PT dashboard compliance card + activity feed (v86)** — filter tabs (All / At risk / Active), compliance rows with `data-zone`, `sessionsThisWeekTotal`, recency colour labels
- **Client dashboard recent sessions tappable (v87)** — `openWorkoutLog` wired to session rows
- **Runner state cleanup (v88)** — removed stale `weightInput`/`repsInput`/`activeField` from `_runner`
- **Settings page (v88)** — profile name edit, change password, account info, sign out
- **Check-in sparklines (v89)** — colour-coded bar charts per metric in `renderClientOverview`
- **Client session history page (v90)** — "SESSION HISTORY" header with count, `#client-session-list`, timezone-safe date rendering
- **Recency labels on client list (v91)** — Today / Yesterday / Xd ago with green/amber/red colour coding
- **Playwright E2E suite (v92–v93)** — 14 tests across 3 spec files; 14/14 passing clean; headless Chromium, 390×844 viewport
- **E2E seed script (v93)** — `node scripts/seed-test-data.js`; creates `coachapp.e2e.pt@gmail.com` + `coachapp.e2e.client@gmail.com` with 6 exercises, 1 template, 5 sessions, 13 weight entries, 1 goal + milestones, 4 check-ins; idempotent
- **Console error capture (v93)** — `tests/fixtures.js` fixture attaches `page.on('console')` + `page.on('pageerror')` on every test; errors surface as HTML report annotations without auto-failing
- **Video on failure (v93)** — `playwright.config.js` now has `video: 'retain-on-failure'`; full `.webm` recording alongside screenshot + trace on any failure
- **Playwright selector fixes (v93)** — flaky `h1` nav check replaced with `waitForSelector('text=START A WORKOUT')`; `#workout-runner` (zero-height fixed container) replaced with `button:has-text("End")` assertion
- **Skills overhaul** — `/save`, `/playwright`, `/deploy-check` created; `hello-claude` rewritten to include all memory rules + standing behaviours; `/save` updated with cache bust check + memory sweep + Playwright status steps; `/deploy-check` updated with `/code-review ultra` as step 2; `/code-review` offer added to pre-push standing behaviour
- **Memory additions** — `project_playwright.md`, `feedback_save_skill.md`, `feedback_deploy_check.md`, `feedback_playwright_skill.md`; `feedback_e2e_testing.md` updated with 4 new Playwright lessons
- **Daily question cron** — fires 9:07am while Claude Code is open; 30 questions in `C:\Users\jaken\.claude\daily-questions\questions.md`; answers saved to log + memory

**Bugs fixed:**
- Playwright "start and cancel" test — was asserting on `text=Start workout` modal that doesn't appear when template is pre-selected; fixed to assert on `button:has-text("End")` which is always visible in the runner
- Playwright selector `'text=All sessions, text=Back'` — invalid compound selector treated as literal string; fixed to `'text=All sessions'`
- `seed-test-data.js` — `template_id` column doesn't exist on `workout_logs`; removed from insert

**Decided:** All Playwright specs must import `{ test, expect }` from `tests/fixtures.js`, never directly from `@playwright/test`.
**Why:** The fixture adds console error capture to every test automatically. Importing directly bypasses it silently.
**Revisit-if:** Never — one-line import change, zero cost.

**Decided:** Daily question cron — session-only despite `durable: true` request (platform constraint).
**Why:** CronCreate returned "session-only" even with `durable: true`. Cron must be recreated if Claude Code restarts. Jake is aware.
**Revisit-if:** If Claude Code adds persistent cron support — switch to durable.

## 2026-06-24 — Live E2E test, bug fixes, mobile dashboard (v79–v80)

**Done:**
- **Dashboard grid responsive (v79)** — changed `grid-template-columns:1fr 1fr` to `repeat(auto-fit,minmax(300px,1fr))`. On mobile the two-column layout was cutting "This week's sessions" off the right edge. Auto-fit collapses to single column at <600px.
- **Full E2E test as client** — ran Push A workout (6 exercises × 2 sets × 80kg × 8 reps) in client view on live site. Session saved correctly; data confirmed in PT session log view.
- **Bug found + fixed: post-save navigation (v80)** — `saveRunnerSession()` always called `openClient(clientId)` after saving, which opens the PT client profile. Clients were dropped on the PT view of their own profile instead of their Workouts page. Fix: `if (currentView === 'client') navigate('workouts'); else openClient(clientId)`.
- **Bug found + fixed: client session history not tappable (v80)** — `renderClientWorkoutsPage` rendered session rows with `cursor:default` and no `onclick`. PT tab view had `openWorkoutLog` wired; client standalone page didn't. Fix: added `onclick="openWorkoutLog(..."` and `cursor:pointer`.
- **FK constraint confirmed present** — `workout_log_sets_exercise_id_fkey` already exists (SQL returned "constraint already exists"). Two-query workaround in `openWorkoutLog` now redundant; can simplify to nested join in future cleanup.
- **dbq() sweep confirmed complete** — all write paths have `if (error)` checks; no silent failures remain.
- **E2E testing lessons banked** — `feedback_e2e_testing.md` created in memory; 4 lessons: test as client not PT; PT/client render functions drift; save functions must not own navigation; E2E before beta.

**UNVERIFIED (banked):**
- Dashboard stacked layout at 390px — v79 pushed live, but couldn't screenshot at mobile width (Chrome renderer kept freezing). Logic is correct (auto-fit is the standard responsive pattern) but not visually confirmed.
- Post-save navigation fix (v80) — pushed but not re-tested end-to-end (runner test would take another full workout run).
- Client session history tappable (v80) — pushed but not tested live.

**Decided:** E2E testing as a client user is mandatory before every deploy that touches the runner or session history — not just code review.
**Why:** Two real bugs were found in 5 minutes of live client testing that passed all 9 pre-push checks. Both were invisible to grep and code review because they were navigation/interaction wiring issues, not logic bugs.
**Revisit-if:** Playwright E2E is set up (Week 2 plan) — at that point the manual run is replaced by automated coverage.

**Decided:** Save functions must not own navigation decisions.
**Why:** `saveRunnerSession` calling `openClient()` directly caused wrong navigation for client users. The save function's job is saving; routing is the caller's concern or must be role-aware at the call site.
**Revisit-if:** Never — rule now banked in STATUS.md bug patterns.

## 2026-06-23 — Memory system expansion (no code changes)

**Done:**
- `user_jake.md` created — user profile: who Jake is, technical level, communication style, "be brief" rule, dual-explanation goal, learning intent
- `feedback_educate.md` updated — now specifies dual-layer format (technical + simple), inline delivery, not just "explain after decisions"
- `feedback_post_build_review.md` created — post-build review triggers automatically after every end-to-end test
- `feedback_skill_creator.md` created — recurring failures must become skill files immediately; never leave orphans
- `feedback_verify.md` created — nothing reported as done without verification; UNVERIFIED banked with reason if check impossible
- `feedback_code_review.md` created — grep scan at session start + /code-review before every deploy; both mandatory
- `MEMORY.md` updated — 5 new entries indexed; 16 total

**Decided:** Bank session behaviours (review, verify, skill creation, code review) as memory entries, not just skill files.
**Why:** Skill files only load when the skill is invoked. Memory loads at every session start. Behaviours that must be automatic belong in memory so they don't require prompting.
**Revisit-if:** MEMORY.md grows past ~20 entries and becomes unwieldy — consolidate at that point.

**Unverified claims sweep:** Nothing built this session — no unverified claims.

## 2026-06-23 — CI pipeline, RLS optimisation, photo transforms, documentation research (v76)

**Done:**
- **Documentation research (5 parallel agents):** Supabase RLS, PostgREST nested queries, Realtime, Storage, GitHub Actions — all researched and key findings extracted
- **`scripts/checks.sh` created** — pre-push checks moved from `.git/hooks/pre-push` (untracked) to a tracked file; git hook now delegates to it so CI and local run identical logic
- **`.github/workflows/deploy.yml` created** — CI runs checks on every PR and push; deploy job only fires after checks pass; closes the `--no-verify` bypass gap. UNVERIFIED — push was rejected because PAT lacked `workflow` scope; Jake needs to add it and push
- **Progress photos** — Supabase image transform params (`?width=600&height=800&resize=cover`) appended to `getPublicUrl` output; photos now auto-resized on fetch, no upload changes needed
- **RLS optimisation** — ran SQL in Supabase: 10 duplicate SELECT policies dropped, all 50+ policies rewritten with `(SELECT auth.uid())` wrapper (~95% per-row speedup per Supabase docs), new `Client inserts own workout log sets` policy added (was missing — client self-logging would silently lose set data)
- **Memory saved** — "always repost code when correcting" rule banked after Jake flagged it

**UNVERIFIED (banked):**
- GitHub Actions CI — push rejected due to PAT `workflow` scope; awaiting Jake to fix token and push
- RLS policy correctness — SQL ran without errors but client-side behaviour not tested post-migration

**Decided:** Move pre-push checks to `scripts/checks.sh` rather than keeping them only in `.git/hooks/pre-push`.
**Why:** `.git/hooks/` is not tracked by git — the hook could be lost on re-clone and could be bypassed with `--no-verify`. A tracked script with CI enforcement closes both gaps.
**Revisit-if:** Never — clean permanent pattern.

**Decided:** `(SELECT auth.uid())` wrapper on all RLS policies as a standard.
**Why:** Postgres re-evaluates `auth.uid()` for every row when called bare. Wrapping it in a subquery tells the planner to evaluate it once per statement — Supabase docs cite ~95% speedup (179ms → 9ms) on policies that do this.
**Revisit-if:** Never — strictly better, no downsides.

## 2026-06-22 — Bug sweep: set_type, clearInterval, race conditions, session summary (v70–v75)

**Done:**
- **`set_type` constraint bug fixed** — `set_type: 'working'` violated `workout_log_sets_set_type_check` DB constraint; all set data dropped silently while session row saved. Fix: removed `set_type` from both `saveRunnerSession` and `saveWorkoutSession` inserts; filter threshold changed from `> 3` to `> 2` (base row now 2 keys after removing set_type). Added `setsHadError` flag so warn toast fires if any sets fail, success toast only if all clean.
- **LOG blocked after Set 1 fixed** — `clearInterval(_runner._restInterval)` left variable as truthy numeric ID; `if (_runner._restInterval) return` guard blocked LOG forever. Fix: introduced `clearTimer` helper (`const clearTimer = id => { clearInterval(id); return null }`); all three timer variables (`_timerInterval`, `_restInterval`, `_intervalInterval`) now use `x = clearTimer(x)` pattern.
- **Pre-push hook extended** — checks 6, 7, 8 added: `set_type:` in inserts, swallowed write errors (`if (setsErr) log.` with no abort), bare `clearInterval()`. False positive in check 7 tuned (excludes SELECT reads and `setsHadError` pattern). False positive in check 8 fixed (`://` exclusion catches comment lines from grep -n output).
- **Race condition fixed** — `saveRunnerSession` now snapshots all `_runner` fields into locals before first `await`; `discardRunner()` mid-save no longer crashes.
- **Stale timer guard** — `_timerInterval` callback gets `if (!_runner) return` guard.
- **5 remaining `alert()` calls replaced** — with `log.error` (auto-toast) or `showToast`.
- **Session summary screen built** — `showRunnerFinish()` shows stats row (total volume, sets, duration), exercise cards, async PR detection with 🏆 badges (two-pass render: immediate then re-render after DB query for previous bests).
- **`_activePage` persistence** — `navigate()` writes to `localStorage._activePage`; `showApp()` reads it; client refresh now lands on last visited page.
- **Research:** Supabase RLS, PostgREST, Realtime, Storage, GitHub Actions docs read by parallel agents. Key findings: `(SELECT auth.uid())` wrapper, Realtime subscription pattern for client notifications.
- **STATUS.md bug pattern list updated** — `set_type`, `clearInterval` without null, swallowed write errors all added.
- v75 committed and pushed.

**Decided:** `clearTimer` helper as the only permitted way to clear an interval.
**Why:** `clearInterval(x)` cancels the timer but leaves `x` as a truthy integer. Any `if (x)` guard then fires forever. The helper returns `null` so the caller can zero the variable in one expression.
**Revisit-if:** Never — the pre-push hook now enforces this pattern.

## 2026-06-22 — Cardio runner fixes, session history, error infrastructure (v63–v65)

**Done:**
- **v63 — Cardio runner fixes:** "Done early — LOG" bug fixed (was reading `#wr-cardio-dur` while interval overlay was open — element doesn't exist; fix: compute elapsed from `intervalSecs - intervalRemaining`); back-navigation in runner added; null guard on session in runner
- **v64 — Session history fix:** `openWorkoutLog` was crashing with PGRST200 — `workout_log_sets` has no FK constraint to `workout_log_exercises`, so PostgREST nested join returned null. Fixed with two separate queries + manual merge. Also renamed `log` → `session` variable (was shadowing the `log` logging utility). Richer template card set preview (cardio now shows pace range, rest window, HR zone, not just duration). Runner now shows "Coach note" callout if exercise has notes. `saveRunnerSession` bare `alert()` replaced with `log.error()`.
- **v65 — Error infrastructure:** `dbq()` wrapper added — auto-logs all Supabase errors; any query wrapped with it will surface PGRST codes in console immediately. Auth re-render guard fixed — `onAuthStateChange` was calling `showApp()` on every `SIGNED_IN` event (including token refreshes), which snapped real users back to PT dashboard mid-session. Fixed to `if (!_appLoaded)` — bootstraps once only. Four unguarded high-risk write sites converted to `dbq()`.
- **Supabase API logs** — added to memory; reminder to check dashboard → Logs → API after each test session
- **FK constraint SQL** — provided for permanent fix of workout_log_sets join; Jake needs to run in Supabase SQL editor

**Decided:** Two-query pattern for PostgREST when no FK exists — fetch parent + children separately, merge in JS.
**Why:** PostgREST requires actual FK constraints to traverse embedded resources. The `workout_log_sets.workout_log_exercise_id` column exists but has no DB-level FK. Adding the FK is the permanent fix; the two-query workaround is the safe interim.
**Revisit-if:** Jake runs the FK constraint SQL — then `openWorkoutLog` can revert to the cleaner nested join.

**UNVERIFIED (banked):**
- `openWorkoutLog` two-query fix — logic correct, not tested end-to-end (preview crashed before navigation)
- Runner "Coach note" callout — pushed v64, not tested in preview

## 2026-06-22 — Bug audit + automated review system + repo cleanup

**Done:**
- **Bug audit (10 bugs reviewed, 7 fixed):** dashboard stats scoped to `coach_id`; weight_logs activity feed fixed (`logged_at` → `created_at`); coach notes display fixed (`log.coach_notes` → `log.notes`); `renderClients` scoped to coach; `renderWorkoutTemplates` scoped to coach; client workouts page now uses `clients.coach_id` not `currentUser.id`; runner templates query resolves coach_id correctly for client users; activity feed client click now uses `openClient(id)` instead of name lookup (breaks on duplicates)
- **Role routing fix:** null-role profiles (invited clients missing `profiles.role`) now correctly inferred from `clients` table and patched on first login — prevents client refresh landing on PT dashboard
- **Pre-push git hook** created at `coachapp/.git/hooks/pre-push` — catches: wrong column names (logged_at on workout_logs/weight_logs, coach_notes), unscoped multi-tenant queries, missing cache bust version, bare alert() calls, hardcoded UUIDs/emails, duplicate function definitions. Errors block push; warnings pass through
- **hello-claude skill updated** — Step 4 is now an automated code review scan
- **"Copy last set" button fixed**
- app.js v=52 committed and pushed to GitHub Pages

## 2026-06-21 — Workout Runner, client Workouts page, master account view switcher

**Done:**
- **Workout Runner built** — full-screen real-time gym logger.
- **Client Workouts page** — clients see templates with ▶ Start buttons + recent sessions.
- **Master account view switcher** — PT|Client pill; auto-detects master account.
- **mobile-check skill created**
- **Jake's client record inserted**
- v=34 deployed to Netlify.

## 2026-06-20 — git, skills, protocols

**Done:**
- git init + initial commit; run-coachapp skill built and verified live; hello-claude global skill created; /save confirmed as built-in; three client header buttons added; slash command reference banked.

## 2026-06-20 — Invite system fix + clean rebuild

**Done:**
- Invite system fully fixed: Edge Function stamps user_id at send time using inviteUserByEmail return value + service role. Redundant triggers/RPCs dropped. handle_new_user rewritten. clients.invited_at added. v=15 deployed.

## 2026-06-20 — Foundations: compliance, RLS, SQL cleanup

**Done:**
- Compliance card, test client cleanup, 11 client RLS policies, broken auth.users policy fixed, header buttons verified, silent failure audit, sql-safety skill created. v=22 committed.
