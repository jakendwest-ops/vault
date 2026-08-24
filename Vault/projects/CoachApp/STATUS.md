# CoachApp — STATUS
_Last updated: 2026-08-24._

> **Session history lives in `LOG.md`, not here.** Until OS v3 this masthead carried a 710-line
> `Previous: … Previous: …` chain going back to 2026-07-19 — every session of it already written up
> in `LOG.md` in structured form. It was deleted 2026-08-23 after verifying each session had a
> `## <date>` entry there and that every distinctive fact (commits `9d0003b`, `57a188a`, `b637e09`,
> `b53dbfc`, `7aeb3ae`, the 390px viewport finding, the `solo_only` lockdown, the `escapeAttr` bug,
> the ~350 purged rows) survives in `LOG.md`. It cost ~70k chars of every single session.
>
> **This file holds LIVE STATE. If you are writing history into it, it belongs in `LOG.md`.**

## Live state

**App version:** app-core v=16 · app-dashboard v=13 · app-clients v=15 · app-programs v=45 · app-calendar-goals v=17 · app-workouts v=77 · app-runner v=75 · app-progress v=52 · starter-content v=5 · main.css v=10 — **all pushed and live as of 2026-08-23 (`857c5e1`).**
> **DESIGN TOKENS LANDED 2026-08-23.** `js/` style literals **1,027 → 256**. Every remaining
> literal is a deliberate exclusion, not a miss: JS-string colours that reach Chart.js on a
> canvas (where `var()` cannot resolve), values with no exactly-matching token, and attributes
> containing `${...}` interpolation the codemod refuses rather than partially converts.
> ZERO VISUAL CHANGE, proven per module by a byte-identical round-trip (`scripts/tokenise-verify.mjs`).
> The ONE deliberate visual change in the whole set is the Progress table header regaining its
> `--surface-2` background, fixed from a `--surface2` typo — awaiting Jake's eyes.
> Branding is now an edit to the `:root` block of `css/main.css`, not a hunt through nine files.
**CSS version:** v=9 (main.css) — `.ts-grid`/`.ts-cell` added 2026-08-05 for the builder set-editor. `--surface-2`/`--bg-accent`/`--text-accent` DEFINED 2026-07-23 (were referenced 54× and defined nowhere)
**✅ PUSHED 2026-08-14 — `origin/master` = `d337418`** (session 1 of 3 on Jake's screenshot feedback; deploy green, `checks.sh` 56 passed / 0 flaky on the push)

**Live verification owed (2026-08-14):** three surfaces Jake should eyeball — the calendar day modal, the
AMRAP/Unilateral pills in the builder + runner, and the 1RM grid. **Specifically: set Settings → weight to
`lb` and open 1RMs.** That is where the review caught a crash, and it should be confirmed by him, not only
by a test.

_Historic:_ **PUSHED 2026-08-12 — `origin/master` = `9d0003b`** (16 commits over two days; deploy green)

_Historic:_ **PUSHED 2026-08-11 — `origin/master` = `b52c5ea`** (11 commits; `b52c5ea` is the post-/deploy-check fix), working tree clean apart from untracked debug
screenshots. `checks.sh` passed on the push (55 passed, 1 flaky). Deploy queued on GitHub Actions.
**Live verification still owed:** the runner changed heavily and is the screen Jake uses mid-session.

_Historic note, superseded:_ **NOT PUSHED — local master ahead of `origin/master` by several commits** (`1ef09c9`, the Solo-genuine-role
merge, plus 2 earlier same-day commits it absorbed). Jake's explicit choice this save — push is tomorrow's
first job, before the SQL migration and live verification (see top of file for the full sequencing). Previous
confirmed-pushed state: `origin/master` = `74d3024`. Latest local: **a CRITICAL confirmed-
exploitable `workout_logs` RLS gap closed (Jake ran the SQL live, confirmed working)**, 3 bugs re-reported
as still-broken root-caused for real, a 19-site PII sweep, and 6 more ownership anchors (see top of this
file). Before that: **6 live runner/builder bugs fixed** — 0-weight handling, jump last-session data, the
3×-reported add-exercise render bug, "Bodyweight" category removal, the rest-timer nav-persistence redesign, and a silent zero-phase
program-assignment failure — plus 3 more real bugs the pre-push review caught in those fixes themselves.
Before that, the **intervals redesign + a major
security sweep shipped live** (see further down this file) — new exercise type, new `phase` column, ~33 XSS sinks
closed including one live CRITICAL unrelated to the feature itself. Before that, the **Playwright viewport bug
fixed live** — pure test-infrastructure, no app behavior changed. Before that, the **units
toggle shipped live**. Before that, the **progress overhaul shipped live** (①–②d capture + ③ display rebuild + B1–B6 analytics + runner "vs last session" block). 31 commits from 07febfa, deployed atomically (two pushes: `a02292e` the overhaul + review dedup, then `95e8e8f` the runner-block-shows-immediately tweak). Multi-agent review (3 angles + verifier) before the first push — 0 blocking, fixed one duplicate `_epley1RM`. CI green both. **NEW LIVE COLUMNS:** `metric_type` (exercises/template/log-exercise), `avg_hr`/`max_hr`/`height_cm`/`side` (workout_log_sets), `resting_hr` (weight_logs) — all migrations run live by Jake.
**Hosting:** GitHub Pages — https://jakendwest-ops.github.io/coachapp — deploy source switched 2026-07-03 from legacy branch-deploy to Actions-only (`build_type: workflow`); see CRITICAL.md timeline for why
**Last push:** 1a5cb72 (2026-08-09) — **in-app "Invite a personal user"** (Edge Function + Settings card,
confirmed live end to end — real invite sent, `role='solo'`, self-referential `clients` row all verified).
Pre-push hook: 56/57 smoke green. Previous same-push-gap, oldest first: 27e6e01 (2026-08-08, Programs
builder day-slot picker pool lazy not eager) → 645820a (2026-08-08, interval runner stale-duration display
fix) → bbc2bc0 (2026-08-08, cardio/interval exercise-finish capture card + quick-prefs popover, 2 new DB
columns `pace_500m_secs`/`stroke_rate_spm`) → 1379c05 (2026-08-09, cardio/interval `metric_type` merge, live
data migration run + verified). Full detail on each at top of this file / in the bug ledger below. Previous
save point: 980d324 (2026-08-05) — **exercise builder set-editor 2-column grid restructure** (closes the
2026-07-22 mobile-scroll report). Pre-push hook: 56/57 smoke green, CI green. Previous: de54bdb (2026-08-01) — **weekly full-file review: closed a stored-XSS cluster in the runner's
prescription render path (~15 sinks), confirmed 2 more tables already RLS-safe, and confirmed (not just
suspected) a solo_only starter-content seeding bug deferred to its own session.** Pre-push hook: 56/57
smoke green. Previous: 2c70f3d (2026-07-30) — **security follow-up: behaviourally probed `workout_log_exercises`/
`workout_log_sets` + `goals`/`goal_milestones` for the same shape of gap as the CRITICAL `workout_logs`
fix — all confirmed already RLS-safe. `saveEditGoal` anchored to match, 3 siblings deliberately left
unanchored (client-reachable or no owner column — RLS already covers them).** Pre-push hook: 56/57 smoke
green. Previous: 74d3024 (2026-07-30) — **CRITICAL workout_logs RLS gap closed (confirmed exploitable, now
confirmed fixed by Jake live) + root-caused 3 bugs re-reported as still-broken + 19-site PII sweep + 6 more
ownership anchors.** See the top of this file for the full writeup.
Previous: eb9ec3f (2026-07-29) — **6 runner/builder bug fixes + 3 review-found fixes.** See the top of
this file for the full writeup. Previous: 60238be (2026-07-28) — **Intervals redesign + a major security sweep.**
For that writeup: new block-model exercise type + phase-walk runner (9-task SDD build, 8 real bugs caught
and fixed), then a mandatory whole-branch review surfaced a live, unrelated CRITICAL client→coach stored-XSS
(`client_check_ins.notes`) and its wider sweep (~33 sinks across 5 commits). New migration
`scripts/add-interval-type-2026-07-25.sql`, run live by Jake. 279 declared (full run) = 273-275 passed / 2
skipped / 0 failed (run alone) — 4-5 recurring failures are a confirmed pre-existing, unrelated
`client_programs` test-data gap, not a regression. (Previous: edb8995 (2026-07-25, 2nd save) — **Fixed the Playwright mobile-viewport bug + the nav-selector
ambiguity it had been masking.** See the top of this file for the full writeup. Pure test-infrastructure — the
only product-code change was one inert `id` attribute (app-progress v26→v27). 220 declared = 218 passed / 2
skipped / 0 failed / 1 flake (reproduced clean in isolation). Multi-agent review (3 angles) clean.
(Previous: 23a2493 (2026-07-25, 1st save) — **Units toggle: account-wide weight/jump-height/cardio-distance preference.**
See the top of this file for the full writeup (scope, the 5 multi-agent-review findings, the test-fixture-pollution
bug found and fixed along the way). New migration `scripts/add-unit-prefs-2026-07-24.sql`, run live by Jake. New
`tests/units-2026-07-24.spec.js` (6 tests). 220 declared = 218 passed / 2 skipped / 0 failed / 0 flaky.
(Previous: 57a188a (2026-07-24, 3rd save) — **Security: public self-signup closed, solo_only lockdown, a live role-corruption incident fixed.** Jake's brother self-registered through the public "Sign up" form and landed on a full PT/coach account — removed the form, its handlers, and the show-signup/show-login toggle entirely (Jake separately needs to flip "Allow new users to sign up" off in the Supabase Auth dashboard, since `db.auth.signUp()` is reachable with just the anon key regardless of UI). Jake then asked for the brother's account to be locked to personal-only with no path to a coach dashboard: new `profiles.solo_only` — when set, `loadUserInfo()` skips `window._masterAccount` entirely so the view-switcher never renders and `switchView()`'s guard blocks any escape. **While building this, a real incident happened**: an edit that added the new `solo_only` column to a `.select()` landed live before Jake's migration had actually run, and a full Playwright suite was already mid-flight against the real Supabase DB — every login in that window hit a missing-column error, and `loadUserInfo`'s pre-existing role-inference fallback (which writes `role:'client'` to the DB whenever it sees a falsy role AND finds any clients row for that user, no `coach_id` filter) silently, permanently downgraded the real PT test account from coach to client in production. Root-caused, repaired, and fixed properly: the fallback now requires `!error` before touching the database. Banked as **les-053: never edit application source while a Playwright run is in flight against the live dev server** — even a change that looks dormant can error an unrelated query and trigger unrelated write-back logic. **Multi-agent review then caught 3 more real bugs in the solo_only work itself, all fixed same push:** the branch force-assigned role:'solo' even when its clients-row lookup came back empty (no fallback, and since switchView is deliberately blocked, no way out); `window._masterAccount`/`_soloClientId`/`_masterClientId` and the switcher's visibility were session state that only ever got SET, never reset on sign-out — the primary sidebar sign-out button doesn't reload the page, so the next account signing in on the same tab (a real shared/gym-device scenario this codebase already designs around) inherited the previous account's `_masterAccount=true`, the one thing `switchView`'s guard checks; and a failed profile fetch defaulted an unrecognised role straight to the full coach nav — `showApp` now fails closed with a retry screen instead. **Separately diagnosed, not a bug**: a LATER 142-failure, 1.5-hour test run turned out to be Supabase Auth rate-limiting from the earlier bad run's retry storm, not a repeat of the corruption — confirmed by a clean, fast re-run once the cooldown passed. **Known, banked limitation**: `solo_only` enforcement is currently client-side only — this session's own repair work already proved a user's own authenticated session can self-write `solo_only`/`role` via the anon client (used repeatedly tonight to fix stuck test-account state), since RLS on `profiles` is row-scoped not column-scoped. Low practical risk for the one real account this affects (a trusted family member); real fix needs a DB-level trigger, tied to the already-banked "Solo becomes a genuine account type" pre-beta decision. 214 declared = 212 passed / 2 skipped / 0 failed / 0 flaky. New tests: `signup-removed-2026-07-24.spec.js` (3), `solo-only-2026-07-24.spec.js` (5), `role-inference-safety-2026-07-24.spec.js` (2). app-core v5→v6, app-progress v24→v25. (Previous: 7aeb3ae (2026-07-24, 2nd save) — **Intervals: get-ready countdown + builder repeat-set×N.** Reading `app-runner.js` before proposing anything showed the work→rest→work loop Jake described already existed and worked (`startIntervalTimer`/`startRestTimer` auto-chain via `_afterRest`; both already speak the last 5 seconds aloud). Built the one real gap: `startRunnerCountIn`/`stopRunnerCountIn`/`renderRunnerCountIn` — a 5-second spoken "5,4,3,2,1,Go!" lead-in before the FIRST work interval, then hands off to the existing unmodified `startIntervalTimer`. `renderIntervalTimer`/`renderRestTimer` now show "Round N of M" instead of bare "Set N" for multi-round cardio. Builder: `repeatTemplateSet(i)` clones a set N-1 more times in one click — Jake's explicit ask scoping the builder side ("shouldn't have to click add>copy 10 times"); type-agnostic, so 5×5 straight sets get the same shortcut as 5 cardio rounds. **Multi-agent review caught a real bug in the fix itself**: `logRunnerSet` already blocks itself during rest (`if (_runner._restInterval) return`) but had no equivalent guard for the new count-in — since the cardio duration field is pre-filled with the target duration, LOG firing mid-countdown would have silently logged an un-performed round; `showRunnerFinish`'s own "FULL teardown" comment (written after a past stray-timer-under-the-finish-screen bug) didn't cover the new timer either. Both fixed same push, red→green verified. Also capped `repeatTemplateSet`'s round count at 50 against a typo splicing hundreds of clones. 204 declared = 201 passed / 2 skipped / 1 flaky (unrelated pre-existing runner-launch race, passed on retry), 0 failed. New `tests/intervals-2026-07-24.spec.js` (7 tests). app-runner v32→v33, app-workouts v33→v34. (Previous: b637e09 (2026-07-24) — **6-bug ledger batch: legacy metric_type, Epley ceiling, real PB picker, distance formatting, discard confirm, cardio LOG toasts.** All six from the 2026-07-23 full-file review. `_resolveMetricType` (shared by runner + builder edit-modal) makes the legacy `sets_json[0].unilateral/.timed` fallback reachable again — was dead code because `launchRunner` always supplied a truthy `metric_type` first. `_estimate1RM` consolidates 4 duplicate Epley formulas with a 12-rep ceiling (was: 60kg×30reps → junk 120kg "1RM" driving every %1RM target). `_bestPerfLog` consolidates 3 sites that badged the newest `performance_logs` entry "PB" regardless of value. 4 distance-format call sites now route through `fmtDistanceM`. Runner Discard button now confirms — scoped correctly to just that one caller (`discardRunner`'s other 2 callers, empty-session-end and post-save-cleanup, stay silent by design; an earlier version of this fix put the confirm in the shared teardown itself and broke both). `logRunnerSet`'s 6 missing-field guards now toast. **Multi-agent review caught a real bug in the fix itself**: `_bestPerfLog` compared raw numbers across incompatible units — `PERF_CATEGORIES` lets the same exercise be logged in kg OR lbs, min OR sec, as a free per-entry choice, so 220lbs (~99.8kg) would beat 100kg purely because 220>100. Fixed with a small `_PERF_UNIT_BASE` conversion table before comparing; also fixed `renderProgressPBs` reading a stale cached group-unit instead of `best.unit` (could show "100 lbs" when neither entry was ever that). Both caught before push, red→green verified. 197 declared = 195 passed / 2 skipped / 0 failed. app-workouts v33, app-runner v32, app-progress v24, app-programs v23. (Previous: b53dbfc (2026-07-23, 2nd save) — **Day rows show the real prescription; picker says WHICH duplicate.** Every day row now reads `4 × 8–10 reps · 100kg · RPE 8 · 2:00 rest` instead of a bare set count, across **all four** surfaces that render that block (client/solo Workouts, Programs builder slot, and the coach's view of a client's plan — the fourth caught by the review). Substance was killing the duplication: the formatting existed TWICE and had drifted, so both copies now call one `_fmtSetDetail` + `_fmtSetsCollapsed`. Picker rows gained `↳ Used in <phase> · Wk N · MON` / `Not used yet` + exercise count, and the two duplicated pool builders collapsed into `_buildProgramTemplatePool`. **Review found 5 blocking:** `_DAY_LABELS` indexed 0-based against a 1-based `day_of_week` (every label named the wrong day, Sunday vanished); the two rewritten innerHTML sinks were UNESCAPED (client→coach stored-XSS shape); and **three regressions from my own merge** — %1RM hidden when a set also had a weight, AMRAP eating the rep floor, the new jump branch dropping RPE/RIR (les-048). 181 declared = 179 passed / 2 skipped / 0 failed / 0 flaky. app-workouts v32, app-programs v22, main.css v7. (Previous: c72eb14 (2026-07-23) — **Exercise-builder overhaul.** Cardio distance in METRES (`distanceM` + shared `_cardioDistanceM`/`fmtDistanceM`; legacy km `distance` read-only, never rewritten), **watts** end-to-end (builder targets → runner input → Progress chip; new `avg_watts smallint`, migration applied live by Jake, clamped at the save site), `Pace / km` retired (legacy-only escape hatch), optional cardio+strength fields behind a `+ More targets` disclosure (**3 cardio sets at 390px: ~1320px → 685px**; weight_reps 1138px → 832px), **jump targets** finally prescribable (height/distance/jumps/rest/RPE + the missing `_buildTargetCols` jump branch — the runner target bar rendered EMPTY for jumps), and the builder repainted from **15 hardcoded greys → 0** after defining `--surface-2`/`--bg-accent`/`--text-accent` (referenced **54×** across 7 files, defined nowhere — an 18-day-old ledger row). Driven by Jake's copycat concern re: Heavyset. **Multi-agent review found 2 pre-existing LIVE bugs:** the ADD path's `cleanSets` allowlist silently discarded every cardio target except duration/distance (edit path kept them — two siblings drifted; now one shared `_cleanTemplateSets`), and **`metric_type` was dropped by BOTH clone paths** so every ASSIGNED plan fell back to `weight_reps`, losing jump/timed/unilateral routing for the person actually training. Plus 7 non-blocking. **Then re-running the suite caught a ReferenceError I introduced while fixing those** (`derived` existed in the builder's scope but never the runner's — froze the Swap/Add modal open; static gates all green because it's a runtime error). Banked as les-047. 177 declared = 175 passed / 2 skipped / 0 failed / 0 flaky. app-workouts v31, app-runner v29, app-progress v21, app-programs v21, main.css v6. (Previous: 95e8e8f (2026-07-19) — **Progress overhaul + analytics SHIPPED LIVE.** The full capture→display feature deployed to master (31 commits from 07febfa): metric_type capture (①–②d) + the Progress→Performance display rebuild (③) + a SetGraph-informed analytics pass. `js/app-progress.js` gained `_metricPointsFor`/`_aggregateSeries`/`_exerciseRecords`/`_diaryExMetrics`/`_setDetailsLine`/`_diaryDelta`/`_buildExerciseSeries` + `_TREND_METRICS`/`_TREND_RANGES`/`_METRIC_COLORS`, a rebuilt `_renderPerfExerciseList` (metric_type trend cards, range selector, records, unilateral dual-line), a rebuilt `renderProgressPerSession`/`_renderPerfSessionDetail` (per-workout tiles + vs-previous deltas + set-details), resting-HR chart in `renderProgressWeight`, and `renderProgressCardio` removed. `js/app-runner.js` gained cardio avg/max-HR inputs + `_runnerVsLast`/`_renderRunnerVsLast` (live vs-last block). `js/app-clients.js`/`app-dashboard.js` gained the resting-HR input on the bodyweight form. 12 new Playwright tests (all TDD red→green), full suite 168/1-flaky/2-skip, multi-agent review 0-blocking (fixed a duplicate `_epley1RM` → renamed `_epleyEst1RM`). **UNVERIFIED:** live-confirmed by Jake only partially (he checked the runner live and asked the block to show from the start — done, redeployed `95e8e8f`); the full analytics on his real data + ④ coach parity still pending. (Previous: 07febfa (2026-07-18) — **Week-tabs redesign (Programs builder + client Workouts page).** One shared model: weeks are tabs (one week on screen at a time), a day/slot opens its workout **inline**, the full-screen session-detail slider is removed from both surfaces, the builder stacks days vertically on mobile (no sideways scroll) and goes multi-column on desktop, and builder slots carry Edit / Remove / **Save to Library** inline. `renderClientWorkoutsPage` weeks→tabs (`_selectReadWeek`, hidden per-week panels) + first phase open by default; `loadAllPhaseWorkouts` renders per-phase week tabs + only the active week (cached in `window._builderWeekData`, persists across add/remove/duplicate/delete, clamps on delete); `renderPhaseWeekGrid` → responsive `.pwk-days` + inline-expand slots (`_toggleBuilderSlot`/`_editPhaseWorkout`, same editor ctx the slider used). app-programs v19→v20, app-workouts v28→v29, main.css v4→v5. Multi-agent review (3 angles + verifier) after commit: no blocking findings; caught the per-workout Save-to-Library button being silently dropped when the slider was removed (restored onto the builder slot; dead drawer branch removed) and a pre-existing raw `sessionSummary` interpolation on the client DAY header (coach→client stored-XSS, now `escapeHtml`'d). Full suite 152 passed / 2 skipped / 0 failed; new `tests/week-tabs.spec.js` (builder + read-page regressions incl. Save-to-Library end-to-end — the read page had **zero** CI coverage before). Live-verified both surfaces at desktop + 390px. **Earlier same session (OS, backed to claude-config):** added an os-lint `stale-predictions` check (RED on any CoachApp prediction past its `verify_by` still `outcome:null`; PTHub excluded) and removed hello-claude's manual "Step 8 — Predictions" — the hook owns prediction staleness now. Proven RED→GREEN on fixtures. (Previous: 90c6d9e (session 26 cont., 6 commits from 8652491) — **(A) 3 program-workflow bug fixes + (B) new-coach starter content.** (A) `4d8813c` — from Jake's real use: (1) assigning a program showed stale "old data until refresh" on Workouts/calendar because `saveAssignProgram`/`saveAssignProgramToClient` fired `_cloneProgramForClient` fire-and-forget then rendered/navigated before the `client_program_workouts` rows existed — now awaited, with a loading state + re-render. (2) "Update all same-named sessions" (`_applyToAllSessions`) **destroyed the whole target workout** (delete-all-exercises + reinsert-source's-full-list, wiping any per-session difference) — replaced with a surgical `_propagateExerciseChangeToTemplates` that applies ONLY the changed exercise, matched by name (the change captured at edit time in `window._lastExerciseChange`). (3) editing a program workout never reached already-assigned copies, so the calendar showed the old version until re-assign — now `_checkClientPlanPropagation` syncs assigned copies of the edited session automatically (the user's own solo copies silently via `_assignedCopiesForSession`; real clients' copies only behind an "Update assigned clients?" confirm). Multi-agent review caught 2 real bugs in the first cut: a stale `_lastExerciseChange` replay when the pre-delete name-fetch failed (now cleared), and a real-client consent bypass in the sibling propagation (now only solo copies sync silently there). app-programs v16→v17, app-workouts v25→v26. (B) `25b978c`/`c8dda75`/`ad7ef80`/`7616d0d`/`90c6d9e` — **empty-app beta blocker SOLVED** end-to-end (brainstorm→spec→plan→implement→review): `js/starter-content.js` seeds ~40 curated exercises + an "Example — Full Body A" workout (6 linked exercises) + an "Example — 4-Week Foundation" program on a new coach's first login, wired into `loadUserInfo`, gated by a new `profiles.starter_seeded` flag (migration applied 2026-07-12; existing profiles backfilled true so nobody is retro-seeded). Not auto-assigned to the coach's calendar; examples labelled "Example —", deletable. Review's one finding — partial-failure stranding (a mid-sequence error left a coach half-onboarded, never retried) — fixed by making the seed FULLY RESUMABLE (each artifact created only if missing, flag flips only when all six present), with a resume regression test. app-core v3→v4, +starter-content v1. Full suite 133 passed / 2 skipped / 0 failed; CI green. Live verification pending: a real new coach signup landing on a populated dashboard. (Previous: 8652491 (session 26, 5 commits from 6e6afb2) — **Security & beta gates.** (1) `8b9bb97` runner polish (committed last session, pushed this one after review). (2) `5e03661` **behavioural RLS audit harness** (`tests/rls-audit.spec.js`) + fixed **Personal Bests**, which had NEVER displayed for anyone: `renderProgressPBs` embedded `performance_exercises` (a table that does not exist), PostgREST rejected the whole query, the error was discarded, page showed "No personal bests logged yet" forever — columns were plain fields on `performance_logs` all along (app-progress v8→v9). Created **Coach B** (`coachapp.e2e.pt2@gmail.com`), the first-ever second coach — cross-coach isolation had never been testable. (3) `a26f0f1` fixed **2 runner bugs the pre-push multi-agent review caught**: `addTableRow` still auto-filled weight+reps from last session (the fix-the-class miss — removed from `_ensureTableRows`, left in its sibling), so "+ Add set" logged a set you never performed; and the wizard still rendered "RPE 8–9" under a column labelled RPE (a verbatim COPY of `_buildTargetCols` whose comment falsely claimed it was shared). Also: `toggleTableSet` now requires a weight (a weightless ticked set silently zeroed volume/PB/ghost-text); the RLS **self-test** was only passing because of stranded fixtures (now plants+cleans its own victim); the seed's workout-history block had **never run** (3 wrong column names) so ghost-text had zero coverage (app-runner v21→v22). (4) `69020b1` **Probe C** — the unexpected-DENY half (s23/s24 class): plants a `client_programs` assignment as the coach, reads it back as the client, asserts every nested embed level resolves. (5) `8652491` **Closed a LIVE cross-tenant storage leak** (see Security) + removed the progress-photos feature (Jake, "for now") + `storage-privacy.spec.js` (app-clients v4→v5, app-progress v9→v10). Full suite 127 passed / 2 skipped / 0 failed; CI green. (Previous: c4b1e67 (session 25, 2 commits from 9f33da0) — (1) `0a3ef1d` Personal/solo **Library** nav page: solo had no route to the Templates/Exercise-Library builder at all (`renderWorkouts` explicitly diverted both `client` and `solo` into the read-only session view), so reusable workouts could only be created inline from a program phase slot, always locked to that one day. Extracted `renderWorkoutLibrary()` from `renderWorkouts()` and gave solo its own `library` nav entry pointing at it — the coach's existing Workouts page is byte-identical, untouched. Opening that UI to solo required a new `workout_templates.is_personal` column (same pattern as `exercises.is_personal`, 2026-07-10): solo and PT share one `coach_id`/`auth.uid()`, so without it a personal template would bleed straight into the PT's real client-facing library and vice versa. Existing 1537 rows default `false` (fix forward, no reclassification). Also fixed a **pre-existing** instance of the same bleed, independent of the feature: `openProgram`'s day-slot picker had no role split either, so solo was already seeing the PT's entire standalone-template pool in that dropdown. 3-agent review + verifier caught 2 real issues before push, both fixed: the `is_personal` carry-over on the 3 clone paths in app-programs.js was a **silent no-op** (source templates are fetched via embedded selects with explicit column lists that omitted `is_personal`, so `tmpl.is_personal` was `undefined` and got dropped from the insert payload — not a live leak, since those rows always carry `client_id`/`generated_from_phase_id` which every `is_personal` read already filters on, but a landmine); and both new fixture tests could orphan `[E2E]` rows on a partial failure (cleanup now by-name and idempotent). (2) `c4b1e67` **Closed a real cross-client RLS leak** on `workout_templates` — see "Security" below. (Previous: 9f33da0, session 24, 9 commits from b126d5b) — RLS audit found 2 real gaps beyond scope: `workout_template_exercises` had NO client-read policy at all (same blind spot as the client_programs bug — solo's shared auth.uid() masked it), breaking exercise-list viewing for every real client; `client_1rms` had write policies for solo but none for a real coached client. Both fixed with new Supabase policies, verified end-to-end against a real client-role account (not just pg_policies). Built runner session autosave (localStorage draft, keyed per client, resume/discard modal, 10s safety-net tick); %1RM target weights now round DOWN to the nearest 2.5kg (single shared function, fixes every display site at once); Delete week button (reuses deleteProgram()'s ownership-aware template-delete check); plate calculator (wizard hint + strength-table target-bar column, deliberately NOT new text under table rows since Jake rejected that pattern there before). Mid-session, Jake found the program picker showing every workout ever created for a program with indistinguishable duplicates — root cause was `openProgram`'s template query missing `.is('program_id', null)`, so one-off "+ Create new workout" templates never left the reuse pool; fixed + relabeled the option "(this day only)" so the workflow is explicit. Exercises table also got an `is_personal` column so Personal-view-created exercises stop bleeding into the PT-facing library and vice versa (found live: "PT account library contains all exercises created on personal account"). 3-agent multi-agent-review before push caught 3 real issues, all fixed: `deletePhaseWeek` could delete a template a sibling week still needed (duplicatePhaseWeek shares template_id until forked-on-edit); resumed drafts skipped the audio/speech unlock gesture (silent rest-timer cues on iOS); runner drafts weren't cleared on sign-out (PII on shared devices). Also ran a full E2E-debris sweep: 73 leftover test programs + 45 test templates deleted via the real `deleteProgram()` function (not raw SQL), verified zero remaining. Full Playwright suite green (116 run, 114 passed, 2 expected conditional skips) both before and after the review fixes; CI green. (Previous: b126d5b — Confirmed the `client_programs` RLS fix (below) works end-to-end and added a dedicated embed-chain regression test; un-fixme'd the 2 tests that had been blocked on it. (Previous: b79c152 — Hotfixed a live crash Jake hit right after the hero-card build shipped: a phase with zero `program_phase_workouts` (a normal state — a phase not yet fully built) crashed `renderDays` on `undefined.forEach`, since pre-existing accordion code assumed at least one week/session always existed. Not part of the day's earlier diff, so the diff-scoped multi-agent review never saw it, and no test fixture (old or new) ever modeled a zero-session phase. Fixed with an explicit empty-phase message; new Playwright regression test added. This same investigation also fully resolved the `client_programs` RLS gap flagged below as "found but not fixed" — Jake applied that policy, then it became clear the first verification pass only checked the outer table: the app's real queries embed `programs(program_phases(program_phase_workouts(...)))` *inside* `client_programs`, and those 3 tables also had no client-read policy, so PostgREST silently nulled the embed and the dashboard still crashed even after the "fix" looked complete. Jake applied 3 more policies; all 4 now confirmed live via a fresh end-to-end fixture test, not just a table check. New standing skill `missed-check-to-test` created from this incident (converts "I checked A but not related B" misses into a Playwright test, same commit as the fix). Full Playwright suite green (91 passed, 2 expected conditional skips, zero remaining `test.fixme`s), pre-push hook green both pushes.) (Previous: 8e9c26c — Finished the Workouts-page hero card + "Recent sessions" rename (interrupted mid-build in session 22 by a process restart). Fixing the resulting Playwright flakiness surfaced two real, pre-existing product bugs, each root-caused rather than patched around: (1) `deleteProgram()` was deleting ANY template referenced by a program's phases with no ownership check — a periodization test's throwaway program had linked the shared seed template into a slot, and deleting that program destroyed it, which is exactly what would happen to a real coach's genuinely reused standalone template. Fixed to only delete templates it actually owns (its own `program_id`, or its own periodization-generated week clones via `generated_from_phase_id`) — a first version of this fix missed the week-clone case (caught by the 3-agent review before push). (2) The Workouts page was unconditionally fetching a heavy templates query (up to 100 rows, nested exercise join) even when a program was assigned and the result was never used — worst-case on the personal/solo account specifically, since it shares the PT account's large historical orphan-template backlog. Now only fetched when actually needed. This is very likely the same root cause as the still-open 2026-07-06 "app runs slow moving to workouts page" report. Also fixed the dead "Log weight" button (same shape as the earlier Log PB fix). Separately **found but not fixed** (needs Supabase dashboard access): `client_programs` has no client-read RLS SELECT policy at all — verified directly that a genuine (non-solo) client account reads back zero rows even completely unfiltered, meaning any real client with an assigned program currently can't see it on their Dashboard or Workouts page. Invisible until now because solo accounts share the coach's own `auth.uid()` and never hit this check. Full Playwright suite green (88 passed, 3 expected skips — 2 conditional + 1 `test.fixme` pending the RLS fix), 3-agent review run before push (caught the week-clone regression above), pre-push hook green (40/41, 1 expected skip). (Previous, session 22, not `/save`d at the time — backfilled here: e600010 — Performance/Personal Bests restructure (client/solo self-view): folded Cardio + 1RMs into Personal Bests; Performance split into "Per session" (most-recent-vs-previous comparison, expand to graph) and "Per exercise" (alphabetical, live-search); moved the Workouts-page 1RM grid into Personal Bests. 3-agent review caught and fixed a stale-cache race between Client/Personal view switches and a Chart.js instance leak on every search keystroke, before push. (Previous: 6d8c6a8 — Fixed 3 confirmed bugs: dead "Log PB" button (form only existed on Dashboard pages, not Progress — same dead-button shape as this session's Log Weight fix); a real solo-mode bug where saving a PB redrew the wrong dashboard; Body Weight "Starting" tile reading the wrong field + a Y-axis clamp requiring both starting AND goal weight to be set. Also hardened `saveEditTemplate`/`deleteTemplate`'s coach_id filter to be role-aware instead of hardcoded (defensive fix, original repro not fully confirmed). (Previous: b1aa50c — Synced the exercise-picker modal's height to `window.visualViewport` instead of a plain `vh` unit. Last session's `height:70vh` fix (682f86f) solved the max-height-tracks-content shrink, but Jake still saw it shrink/drift toward the bottom on his own phone once the on-screen keyboard opened — root cause: `vh` is sized against the layout viewport, which most mobile browsers don't shrink when the keyboard opens, so the box could end up partly hidden behind the keyboard. `visualViewport` tracks the actual visible area; the modal now syncs to it on open and on resize (keyboard show/hide), falling back to the original `vh` values on unsupported browsers. Confirmed via AskUserQuestion that the shrink only happens once the keyboard is up (not before), which is what pointed at this root cause rather than a repeat of the CSS-sizing bug. Self-reviewed only (no subagent multi-agent-review this round, by explicit call given a tight usage budget); `runner.spec.js` 26 passed/2 flaky (pre-existing login-timeout race, unrelated), full pre-push suite 39/39 (1 skipped). **Still UNVERIFIED on Jake's own phone** — Playwright cannot simulate a real on-screen keyboard. (Previous: 298d88d — Fixed solo mode landing on a broken "fetch failed" screen after finishing a workout in the runner. Root cause: `_afterRunnerSave(clientId)` only special-cased role `'client'` (navigate to Workouts); `'solo'` fell into `else openClient(clientId)`, a PT-only function that queries `clients` scoped by `.eq('coach_id', currentUser.id)` — but a solo client record has `coach_id = NULL`, so the query always returned 0 rows and errored. Found via this session's hello-claude code review (not reported by Jake) — the same "role/isClientPlan gate missing a solo branch" bug class the ritual watches for, just on a plain role check instead. Fixed by adding `'solo'` to the working branch; added a new Playwright regression test; multi-agent review (security/scoping, solo-mode, duplicates/regressions) clean; 39/39 Playwright (1 skipped). (Previous: 682f86f — Fixed the exercise picker modal shrinking/drifting toward the bottom of the screen as search results narrow. Root cause: the modal used `max-height:85vh` with no fixed `height`, so its box size tracked its content; on mobile the overlay is bottom-anchored (`align-items:flex-end`), so as fewer results matched a typed query the whole box — search input included — visually shrank and slid down toward the bottom of the screen, often crowding into the on-screen keyboard. Fixed with a fixed `height:70vh`. Verified live at 390×844: modal height held constant (590.8px) across empty/no-match/cleared-query states. Full Playwright suite green (69 passed). (Previous: 444d0f3 — Fixed slow workout save + slow Workouts-page load. Root cause of the slow save: `saveRunnerSession`/`saveWorkoutSession` inserted one exercise row + one sets-batch per exercise, sequentially — up to ~26 sequential network round trips for a 6-exercise session. Both rewritten to batch all exercises into one insert, then all sets into one insert, correlated by `order_index` (not response array order, which PostgREST doesn't guarantee); `saveWorkoutSession` also gained rollback-on-failure, which it never had before, and its per-exercise `_resolveExerciseIdForSave` lookup was parallelized (de-duped by name first to avoid a create-race on repeated exercise names). Measured live before/after on a 6-exercise save: 14 requests/4.7s → 4 requests/1.1s. Root cause of the slow Workouts-page load: both `renderWorkoutTemplates`/`renderClientWorkoutsPage` queries had no explicit `.limit()`, silently riding the global 200-row server cap — added `.limit(100)` to both. A live diagnostic confirmed the historical orphan-template backlog was 103 rows (not ~993 as previously estimated in this file — that figure was never actually measured), 0 of them in use anywhere; Jake ran a safety-checked cleanup SQL and deleted all 103 plus their exercise rows, confirmed 0 remaining. Separately, Jake asked to also clean the shared exercise library down to only what's used by his personal/solo account (explicitly accepting this would also remove exercise-library entries his real clients' templates/logs currently reference, reverting those specific links to name-based matching, not deleting their actual logged history) — SQL provided (detach FK links, then delete), **outcome not yet confirmed back — see open to-dos.** Also found and fixed a real bug mid-session: a stray character corrupted `app-runner.js`'s first line during editing, silently breaking the whole script (traced via a live browser console capture showing a page-load `SyntaxError`, not guessed); fixed, full suite reverified green. 3-agent review caught one real gap in the new rollback test (its cleanup depended on the rollback-under-test actually succeeding) — fixed with unconditional try/finally cleanup. 69/69 Playwright, 3-agent review clean after fixes. (Previous: 31698fe — Fixed the real root cause of that session's test-suite flakiness: `tests/helpers.js`'s `loginAsClient` returned immediately after login instead of waiting for the client dashboard to finish rendering (same async-render race `loginAsPT` already guarded against with its own wait). A test's first click could land before the page/nav had settled and get silently overwritten — this, not "system fatigue," was why client/solo tests were failing intermittently and sometimes permanently after hours of runs. Added the missing wait; confirmed stable across 2 full back-to-back 38-test smoke runs plus a clean run inside the pre-push hook itself. No app code touched, no cache-bust needed. (Previous: 1526704 — Exercise identity linking + new Exercise Picker (app-programs v9→10, app-progress v4→5, app-runner v13→14, app-workouts v12→13): real `exercise_id` FK added to `workout_log_exercises`/`client_1rms` (previously exact free-text name matching, which is why previous-session/1RM data went missing when the same exercise was typed slightly differently). One-time migration backfilled 4,777 template exercises/27 logged exercises/18-19 1RMs; Jake reviewed his exercise list to merge known duplicate spellings. New shared Exercise Picker (search-as-you-type, "Create new exercise", collapsible archived section) replaces the old dropdown+free-text entry in the workout builder, runner swap/add, and 1RM entry; archive/unarchive added to the Exercise Library page. 5 bugs found and fixed via multi-agent review: search-box race condition, 2 missing RLS policies (clients had zero access to `exercises`), an apostrophe-escaping bug breaking names like "Farmer's Carry", 2 double-tap duplicate-entry races. 3-agent review clean after fixes. First push attempt blocked by severe Playwright-suite environmental flakiness after 3+ hours of continuous test runs — required a full environment reset before the retry succeeded. **Session was not `/save`d at the end — LOG.md entry backfilled after the fact from the commit message, see LOG for detail and the honest caveat about missing real-time tracking.** (Previous: 9b1fb9c — pushed session 17's Areas 1/2/4 backlog; multi-agent review before push caught 2 real bugs in that work and fixed them pre-push: the weight-goals form was only reachable from the PT-facing client-profile Weight tab, not the client/solo's own My Progress page — ported there too; `saveRunnerSession`'s failure path left an orphaned partial `workout_logs` row on retry — added rollback. 69/69 Playwright, 3-agent review clean. Previous: 313bc74 — Scoped 5 `clients` queries in app-clients.js (openClient, renderClientOverview, saveUpdateEmail, showEditClientModal, saveEditClient) by `coach_id` in addition to `id` — defense-in-depth, RLS already blocked cross-tenant access; 2-agent review confirmed all 5 are PT-only code paths so no client/solo flow could break. Also tightened the `client-workout.spec.js` "session history" locator to be onclick-scoped like its siblings. Along the way, corrected a stale fact in roadmap.md: solo accounts' own client record has `coach_id = NULL`, not `coach_id = auth.uid()` as previously documented — verified against `app-core.js:132`. (Previous: 84f9267 — Runner exercise-picker freeze fix + silent rest-timer beep fix. Root cause: Swap/Add exercise shared one hardcoded modal id (`add-to-template-modal`); a fast double-tap during the picker's own fetch latency opened two overlays, and `getElementById`/`closeModal` only ever operate on the first — the visible modal could never be closed. Found live during Jake's real gym session 2026-07-04 (personal account) — forced a reload that lost the entire in-progress workout (confirmed no partial/orphaned DB rows resulted, since `saveRunnerSession` was never reached). Fixed with a pending-flag guard + button-disable while the fetch is in flight + error handling that didn't exist before. Also replaced the 5-second tone-beep countdown with spoken numbers across all three timers (rest/timed-set/cardio interval) — hypothesis: iOS suspends the Web Audio API on screen-lock/backgrounding mid-rest far more readily than the Web Speech API the working "10 seconds" voice cue already uses. 3-agent review caught a real bug in the fix itself (cardio interval timer never primed speech synthesis on its own gesture — would have made its new spoken countdown silently never fire); fixed same session. New Playwright regression test reproduces the exact double-tap race. 60/60 Playwright green (some run-to-run flakiness observed, confirmed environmental/network-timing, not code-related — a different unrelated test flakes each run and passes on retry). (Previous: 730738a — Duplicate week (phase card "Duplicate week" button copies a week's day/workout assignments into the next empty week as real independent `program_phase_workouts` rows; every week — 1, manually duplicated, or periodization-generated — is now equally editable via one unified `renderPhaseWeekGrid`, replacing the old week-1-only editable grid); fork-on-edit for shared workout templates (`_resolveEditableTemplateId`/`_cloneSharedMasterTemplate` in app-workouts.js — editing a template through a phase slot now clones it first if that template is referenced by more than one `program_phase_workouts` row, so a rename/edit via one slot never silently mutates a template reused elsewhere, e.g. the same template assigned to both Monday and Tuesday); `deleteProgram()` fix (a solo user's own self-assignment no longer counts as a blocking client; the PT-facing block toast now names the actual assigned clients instead of just a count). 3-agent review (security/scoping, solo-mode, duplicates/render-safety) found one real issue — the "Generate weeks" confirm dialog's wording only mentioned "previously generated" weeks, stale now that weeks 2+ can hold manual content too — fixed same session. 3 new Playwright tests added (59/59 green). Live-verified in preview against real Supabase data (not just Playwright) before push. (Previous: ec30ebf — trivial empty verification commit confirming coachapp pushes still work after the PAT security cleanup; no app code. 609d5ee — removed dead graphify build-tooling; no app code.)))))
**Supabase project:** avilxuiacmtgeoxxhfhc (eu-west-1, Ireland)

### ✅ Progress overhaul (SHIPPED LIVE 2026-07-19 — pushed 95e8e8f; ④ coach parity remains)
A capture→display feature so the Progress page tracks every exercise category over weeks/months/years.
Design: `docs/superpowers/specs/2026-07-18-progress-tracking-overhaul-design.md`. Plans under `docs/superpowers/plans/` (incl. `2026-07-19-progress-display-rebuild.md`, `-progress-manual-hr-inputs.md`; the analytics build followed the plan `~/.claude/plans/ingest-most-recent-from-glimmering-treasure.md`). **①–③ + the B1–B6 analytics + runner block are LIVE.** ④ coach parity NOT built.
- **✅ ① Data model** (`scripts/add-metric-type-2026-07-18.sql`, run live by Jake): `metric_type` on `exercises`/`workout_template_exercises`/`workout_log_exercises` (default `weight_reps`, CHECK-constrained to 6 values); typed `avg_hr`/`max_hr`/`height_cm`/`side` on `workout_log_sets`. Backfill: cardio from `exercise_type`, jumps by name; NO cardio name-guess on the library (review caught it mislabeling Barbell Row/Cable Crunch — dropped). Verified live.
- **✅ ②b Save-persistence** (app-runner v24): `saveRunnerSession` now persists unilateral (as two `side` rows), timed, distance, HR, and stamps `metric_type` on the logged exercise — was silently dropping all of it. Round-trip test.
- **✅ ②a Builder picker** (app-workouts v30): the Strength/Cardio dropdown → a 6-option `metric_type` picker driving the set-target fields; save derives `exercise_type` + per-set `unilateral`/`timed` flags from it (runner bridge) + remembers metric_type on the exercise. Supplementary backfill (`scripts/backfill-metric-type-flags-2026-07-18.sql`, run live: 23 timed_hold, 0 unilateral). **6-type model, amrap dropped → per-set flag (spec revised).**
- **✅ ②c Adaptive fast table** (app-runner v25): the fast table renders columns per `metric_type` (weight_reps / unilateral L-R rows / timed / jump height+distance); wizard retired for those (cardio only); add/swap carries metricType. 8 stale runner tests updated (wizard→fast-table via a `logTableSet` helper).
- **✅ ②d Manual HR** (app-runner v24→26 for cardio avg/max-HR inputs + resting HR on the **bodyweight form**, not check-in — the check-in form is client-dashboard-only so solo couldn't reach it; `resting_hr` on `weight_logs`, migration `scripts/add-resting-hr-2026-07-19.sql` run live). app-clients v7, app-dashboard v5.
- **✅ ③ Display rebuild + analytics** (app-progress v11→20, app-runner v26→28): metric_type-aware trend cards for every type + Personal Records block + range selector (B1/①); **Intensity (kg/rep)** metric (B2); Recent-sessions **diary** upgrade — per-workout tiles + per-metric vs-previous deltas + set-details line, Per-exercise now default (B3); resting-HR trend on Body tab (B4); Cardio-bests section removed (B5); our own metric colour palette (B6); live runner **"vs last session"** block (C, shows from the moment you reach a strength exercise). SetGraph reference ingested into the wiki; anti-plagiarism held (mirror data not design).
- **⬜ ④** coach parity — factor the trend view to `(clientId, role)` and render it read-only in the coach's client-profile too (`renderClientPerformance`/`renderClientWeight` still show the OLD view). The resting-HR input + chart are self-view only; the coach's client-profile weight form doesn't surface them yet. **The queued fast-follow.**

---

## What's working (verified)

- **Copy program workouts → Library** — the missing bridge. A workout built with "+ Create new workout (this day only)" carries `program_id`, so it's deliberately excluded from the reuse pool (the 2026-07-10 clutter fix) and previously could only be reused by retyping it. Now: a "Save to Library" button in the session-detail drawer (per-workout) and "Copy workouts to Library" on the program page (bulk). Reuses `_cloneSharedMasterTemplate(tmpl, overrides)`. **Idempotent** — a same-name library workout is skipped, so clicking twice is safe. app-workouts v25, 2026-07-11.
- **Tap-row workout picker** — the native `<select>` day-slot picker is gone (`_openWorkoutPicker`, modelled on `_openExercisePicker`). An `<option>` can only hold plain text, which is precisely why three "Upper Body" workouts were indistinguishable at the point of assignment (Jake's question). Each row now shows **name + description + exercise preview**. Also closes two long-open complaints about this control: no visible feedback until opened, and an unmanageably growing list. app-programs v16, 2026-07-11.
- **Duplicate week auto-extends the phase** — was hidden entirely on a 1-week phase (`canDuplicateAny = maxWeek < durationWeeks`) with no explanation. Growing the phase IS what duplicating its last week means. `duration_weeks` is bumped **after** a successful insert (bumping first left a failed copy claiming an empty week, silently lengthening an assigned client's program). app-programs v16, 2026-07-11.
- **CRITICAL data-loss bug fixed — Duplicate week → Generate weeks destroyed the Week-1 workout.** See LOG for the full reproduction steps. `_cleanupPhaseWeeksBeyond` deleted every template a stale week referenced with **no ownership check and no still-referenced check** — while its sibling `deletePhaseWeek` had both (added 2026-07-10). Proved red/green (reverting the fix makes the new test report `survived: 0`). Both now share `_deleteOwnedUnreferencedTemplates`. app-programs v16, 2026-07-11.
- **Personal/solo Library page** — solo now has its own `library` nav entry (7 items) routing to the same Templates + Exercise Library builder the coach has (`renderWorkoutLibrary`, extracted from `renderWorkouts`; the coach's Workouts page is unchanged). This is where reusable workouts get built to assign into Programs — solo previously had **no route to it at all**, so its only way to create a workout was the inline "+ Create new workout (this day only)" dropdown in a program phase, which locks the template to that single day slot. app-core v3 / app-workouts v24, 2026-07-11.
- **`workout_templates.is_personal` split** — new boolean column (mirrors `exercises.is_personal` from 2026-07-10). Solo and PT share one `coach_id`/`auth.uid()`, so without this a template built in Personal view would bleed straight into the PT's real client-facing Templates list and vice versa. Applied to all 4 standalone-template read sites + `saveNewTemplate` + the 3 clone paths. Existing 1537 rows default `false` (fix forward, no reclassification — Jake's standing call). **Note: this is a UI display split, deliberately NOT enforced at RLS — see Security below.** 2026-07-11.
- **Program day-slot picker respects the PT/Personal split** — `openProgram`'s template query had no role split either, so solo was already seeing the PT's entire standalone-template pool in that dropdown. Pre-existing bug, independent of the Library feature, fixed alongside it. app-programs v15, 2026-07-11.
- **SECURITY — cross-client RLS leak on `workout_templates` closed** — the `Client reads workout templates` SELECT policy scoped by `coach_id` **alone**, with no `client_id` restriction. Client-plan clones are written with `coach_id = the coach` and `client_id = the client` (`_cloneTemplateForClient`), so **every client of the same coach matched that policy and could read every OTHER client's personalised template clones** via a direct API call. Reproduced live before fixing (logged in as a real client, read back another client's clone by id — red/green, not reasoned). Policy now also requires `client_id is null`; a client's own clones still come through the separate `client_read_own_templates` (client_id-scoped) policy, so nothing legitimate was lost. Also switched the scalar `=` subquery to `in` — it errors outright for any user with >1 `clients` row, which **the master account has** (one coached + one solo record); it only worked because the coach policy matched first. `exercises` needed no change (no `client_id` column, so no cross-client leak is possible there). 2 new regression tests against a **real client-role account** (not solo, which shares the coach's `auth.uid()` and masks exactly this bug class). 2026-07-11, c4b1e67.
- **Runner session autosave** — localStorage-only draft (keyed `_runnerDraft_<clientId>`), checkpointed on every render + a 10s safety-net tick, same-day staleness cutoff, resume/discard confirm modal. Cleared on discard, on successful save, and on sign-out (PII hygiene). Fixes the 2026-07-04 live incident (a runner freeze forced a reload mid-session, wiping the whole workout). app-runner v20, 2026-07-10.
- **%1RM target weights round down to the nearest 2.5kg** — single shared function (`_calcWeightFromPct`) feeds the strength-table target bar, the row pre-fill placeholder, and the wizard's live preview/placeholder, so the fix applies everywhere at once. app-runner v18, 2026-07-10.
- **Delete week button** on a phase — removes a week's sessions + owned templates, any client-propagated copies, renumbers later weeks down by 1, decrements duration_weeks. Reuses `deleteProgram()`'s ownership-aware template-delete check (won't destroy a template a sibling week still points at). app-programs v14, 2026-07-10.
- ~~**Plate calculator**~~ — **REMOVED 2026-07-11 (app-runner v21).** Shipped 2026-07-10 (v19) after "repeated requests" surfaced in the 2026-07-02 competitor research; Jake then used it in a real gym session and asked for it gone — in practice it was noise, not help. Deleted outright (`_calcPlateBreakdown`/`_updatePlateBreakdown`/`_PLATE_SIZES`, the PLATES/SIDE target-bar column, the wizard hint, and its 5 tests). **Worth remembering: the research said build it, the gym said remove it.**
- **Program picker no longer clutters with one-off templates** — `openProgram`'s template query now excludes program-owned templates (`.is('program_id', null)`), so a template created inline via "+ Create new workout" for one day slot doesn't stay in the reuse pool for every other slot/program forever. That option is now labeled "(this day only)" to make the workflow explicit; a real reusable template is built once in Workouts → Templates. Found live (Jake: "every workout that is created in the program appears in this list... impossible for a PT to know which workout is which"). app-programs v13, 2026-07-10.
- **Exercises table separates PT and Personal** — new `is_personal` boolean column; exercises created in Personal view no longer bleed into the PT-facing Exercise Library and vice versa. Found live (Jake: "the exercise library for PT account contains all exercises that have been created on personal account"). Existing (pre-fix) exercises stay as PT's, by Jake's explicit call — only new creates are separated going forward. app-workouts v22, 2026-07-10.
- Auth — login / signup (with consent checkbox) / invite accept / session persistence
- PT dashboard — stats row, recent activity, compliance cards, goals due; business name in subtitle when branding set
- Client management — list, add, profile tabs (Overview / Goals / Workouts / Weight / Performance / Programs / Photos / 1RMs), edit, invite, resend
- Workout templates — create / edit / delete / add exercises / log
- Set form — AMRAP, Unilateral, Timed, Reps, Weight, %1RM, Rest, RPE/RIR, Tempo, Notes
- Cardio set form — Pace, HR Zone, Rest, Stroke rate, Duration, Distance
- Template card set preview — correct for all set types including timed (1:30 not 90 reps)
- Workout runner — set-by-set flow, timed sets, unilateral L/R, rest timer, skip rest, last session strip
- **Strength table (Hevy-style)** — plain-strength exercises show an all-sets-visible table (SET/KG/REPS/✓) instead of the one-set wizard; tap ✓ to complete a set (fires rest timer, non-blocking — table stays visible/editable underneath); uncheck to undo; "+ Add set" appends a row; free-edit any row any time, no forced order. Cardio/timed/unilateral exercises stay on the existing wizard (phase 2). v1, shipped 2026-07-02 (6e6402a). **⚠️ Two v1 choices REVERSED 2026-07-11 (app-runner v21) after real gym use — do not reintroduce them:** the `PREVIOUS` column is **gone** (last session is now **ghost text** in the KG/REPS inputs, so each value sits under the column it belongs to rather than squashed into one 54px cell), and **pre-fill / "1-tap repeat" is gone** (rows start EMPTY — a pre-filled value is indistinguishable from one you actually typed, so a set could be ticked off without the weight ever being confirmed; logging accuracy beat tap-count). **2026-07-03 (v7):** target-info bar restored above the table (reps/RPE-RIR/tempo/rest, or %1RM→kg target when a 1RM is known — `_buildTargetCols`/`_renderTargetBarHtml`); rest timer now renders **inline** in that section as a plain 0:00 countdown instead of a fixed bar covering the runner header; unchecked ✓ button now has a visible outline
- **Mid-workout swap / add exercise (session-only)** — runner header has "⇄ Swap exercise" and "+ Add exercise" (`showExercisePicker`); swap replaces the current exercise in-memory and refetches its previous-session data under the new name; add appends a new exercise and jumps to it. Neither writes to `workout_templates` — today's log only. Picker modal forced to z-index 1000 to sit above the runner's fullscreen layer. v7, 2026-07-03
- **Voice cue fixed** — `_unlockSpeech()` now does a real gesture-tied `speak()` to prime the engine (was only calling `cancel()`, which never registered the gesture, so the "10 seconds" cue silently never fired); `speakCue()` selects a female voice. v7, 2026-07-03
- **Create-new-workout auto-assigns to its day slot** — building a template via the phase grid's "＋ Create new workout" now inserts the `program_phase_workouts` row back into the originating slot instead of leaving it empty. app-programs v6, 2026-07-03
- **Edit from the session-detail drawer** — the read-only session preview now has an Edit button (hidden for `client` role) that hands off to the full template editor with context, so the propagate-to-all-sessions prompt still fires. app-workouts v8, 2026-07-03
- **Phase card shows the periodization range** — e.g. "Linear (70→80%)" / undulating tier %s, instead of just the type name. app-programs v6, 2026-07-03
- **Runner exercise-picker freeze fixed** — Swap/Add exercise can no longer spawn two overlapping modals via a fast double-tap during the picker's fetch window; buttons disable while loading, unique-guard flag prevents the race, error handling added. Rest/timed-set/cardio-interval 5-second beeps switched from Web Audio tones (silently suspended by iOS mid-rest) to spoken numbers via the already-reliable Web Speech channel. 2026-07-04, pushed 84f9267.
- Edit logged sets in runner; PT notes always shown; client notes textarea
- Runner save lands on correct page per role (client → Workouts, PT → client profile)
- Client dashboard — hero "Up next" card (program + phase + week), goals, weight, upcoming events, PBs, recent sessions; PT branding header when set
- PT dashboard — hero card, stats strip (now stacks to 2-across on mobile, 2026-07-05), two-column grid, activity feed (coach-scoped, no solo bleed), compliance cards, goals due
- Solo dashboard — hero card, stats strip (now stays visible on mobile instead of vanishing, 2026-07-05), two-column grid
- **Dashboard CSS consistency pass** — `.dashboard-card`/`.card-header`/`.card-title` now have real CSS definitions (previously undefined, rendered with no background/border/shadow across all 3 dashboards); shared `.dashboard-split-grid` class replaces 3 duplicated inline `<style>` blocks; hardcoded hex colors replaced with design tokens. 2026-07-05, pushed 313bc74.
- **Dashboard "Current program" header** — client + solo dashboards now show a small header (program name + "View program" button) above the "Up next" hero card, when a program is assigned. app-dashboard v3, 2026-07-05, pushed 9b1fb9c 2026-07-06. **Still UNVERIFIED by Jake against his own live account.**
- **Runner %1RM exercises now use the strength table** — `_isPlainStrengthExercise` no longer excludes exercises with an `intensityMin` (%1RM) set; the table's target bar already computed the %1RM→kg display, so no other change was needed. Table row weight input's placeholder now shows the calculated suggested kg when a 1RM is known. Timed/unilateral/cardio exercises still use the wizard (out of scope this session). app-runner v13, 2026-07-05, pushed 9b1fb9c 2026-07-06. **Still UNVERIFIED by Jake against his own live account (Trap Bar Jump).**
- **Weight goals (Starting/Goal weight)** — new small form on the Weight tracking page; two new nullable `clients` columns (`starting_weight_kg`, `goal_weight_kg`) + a new client-role UPDATE RLS policy (first time a client can write to their own `clients` row — every prior UPDATE was PT-only). When both are set, the Body Weight chart's Y-axis uses them as min/max (0.5kg step ticks); falls back to auto-scaling if either is unset. SQL run by Jake, policy confirmed live via `pg_policies`. app-progress v4, 2026-07-05. **2026-07-06 fix:** was only wired into the PT-facing `renderClientWeight`, so clients/solo could never reach it from their own My Progress page (`renderProgressWeight`) — ported there too, and fixed a Y-axis inversion bug for weight-gain goals (calc assumed goal < starting). Pushed 9b1fb9c. **Still UNVERIFIED by Jake against his own live account.**
- **Cardio fields in the workout-preview slider** — `openSessionDetail` now branches on `exercise_type === 'cardio'`, reusing the same formatting already used by the template-card preview (pace/distance/HR/stroke rate); previously fell into the strength branch and showed blank/"—" for every field. app-workouts v12, 2026-07-05, pushed 9b1fb9c 2026-07-06.
- **Update-1RM modal now renders correctly** — root cause was `.modal-box` (used by 5 modals across app-progress.js/app-runner.js) having zero CSS definition anywhere; swapped to the existing, correctly-styled `.modal` class. Also added a `z-index:1000` override to the mid-workout "set your 1RM" sheet (`showRunnerOneRMSheet`), which opens over the runner's own z-index:300 layer and had no override before — UNVERIFIED live, reasoned from the same pattern already fixed for the exercise picker. app-progress v4 / app-runner v13, 2026-07-05, pushed 9b1fb9c 2026-07-06. **z-index fix still UNVERIFIED live.**
- **Runner fixes** — rest time entered on Swap/Add exercise now actually overwrites the original (was silently falling back to 90s); delete-set button has deliberate spacing from the complete-set tick; mobile RPE/RIR label no longer repeats between header and field; "Save workout" no longer shows a false-positive "Save failed" toast when a non-blocking client-lookup hiccups (found in 3 places: `saveRunnerSession`/`saveWorkoutSession`/`showLogSessionModal`); a real stuck-Save-button bug fixed alongside it, plus a second real bug found in review: the exercise-insert failure path left an orphaned partial `workout_logs` row on retry — now rolls back before allowing retry. app-runner v10→v13, 2026-07-05/06, pushed 9b1fb9c.
- **Exercise identity linking + new Exercise Picker** — real `exercise_id` FK on `workout_log_exercises`/`client_1rms` (`workout_template_exercises` already had one) replaces exact free-text name matching, which was silently losing previous-session/1RM history whenever the same exercise was typed slightly differently. Name-match fallback covers older/unlinked rows. One-time migration linked 4,777 template exercises, 27 logged exercises, 18/19 1RMs (Jake reviewed his exercise list to merge known duplicate spellings). New shared Exercise Picker (search-as-you-type, "Create new exercise", collapsible archived section) replaces the old dropdown+free-text entry in the workout builder (add + edit), runner swap/add, and 1RM entry; archive/unarchive added to the Exercise Library page. app-programs v10, app-progress v5, app-runner v14, app-workouts v13, pushed 2026-07-06 (1526704).
- **Workout save is ~4x faster** — `saveRunnerSession`/`saveWorkoutSession` were doing one exercise-insert + one sets-insert per exercise, sequentially (up to ~26 round trips for a 6-exercise session); now batched into 2 inserts total, correlated by `order_index`. `saveWorkoutSession` also gained rollback-on-failure (never had one). Measured live: 14 requests/4.7s → 4 requests/1.1s on a 6-exercise save. app-runner v15, pushed 2026-07-07 (444d0f3). **UNVERIFIED by Jake in a real gym session** — automated verification only so far.
- **Workouts-page queries capped at `.limit(100)`** — were silently riding the global 200-row server cap on every load; now explicitly bounded. Historical orphan-template backlog (confirmed 103 rows, 0 in use) deleted. app-workouts v14, pushed 2026-07-07 (444d0f3).
- **Exercise picker modal no longer shrinks/drifts as search results narrow** — was `max-height` with no fixed `height`, so the box (bottom-anchored on mobile) shrank and slid toward the bottom of the screen, crowding the on-screen keyboard, as fewer results matched. Fixed with `height:70vh`. Verified live at 390×844 (constant height across query states). app-workouts v15, pushed 2026-07-07 (682f86f). **2026-07-08 follow-up:** Jake still saw it shrink on his own phone once the keyboard was actually up — root cause was `vh` units not accounting for the mobile keyboard eating into the layout viewport; modal now syncs its height to `window.visualViewport` instead. app-workouts v16, pushed 2026-07-08 (b1aa50c). **Still UNVERIFIED on Jake's own phone** — Playwright can't simulate a real keyboard.
- **Solo mode no longer breaks after finishing a workout** — `_afterRunnerSave` only special-cased role `'client'`; solo fell through to a PT-only `openClient()` call that queried `coach_id = currentUser.id`, but a solo client record has `coach_id = NULL`, so it always errored. Added `'solo'` to the working branch (navigates to Workouts, same as `'client'`). Found via code review, not a Jake report. app-runner v16, pushed 2026-07-08 (298d88d). Verified via a new Playwright regression test (passing).
- **Exercises-library cleanup confirmed** — Jake ran the cleanup SQL (detach FK links on other clients' rows, then delete unused exercises); confirmed 51 `remaining_exercises` for the coach account. 2026-07-08.
- **Dead "Log PB" and "Log weight" buttons fixed** — both were wired to a form/container that only existed on Dashboard pages, not the Progress page's Body Weight/Personal Bests tabs they're actually clicked from (same bug shape, found and fixed in two separate sessions). Also fixed the corresponding save functions redrawing the wrong dashboard for a solo user. 6d8c6a8 (2026-07-08), 8e9c26c (2026-07-10).
- **Body Weight "Starting" tile + Y-axis clamp fixed** — was reading the wrong field, and the clamp required both starting AND goal weight to be set instead of gracefully falling back. 6d8c6a8, 2026-07-08.
- **Performance/Personal Bests restructure (client/solo self-view)** — Cardio + 1RMs folded into Personal Bests; Performance split into "Per session" (most-recent-vs-previous comparison, expand to graph) and "Per exercise" (alphabetical, live-search); the Workouts-page 1RM grid moved into Personal Bests. e600010, 2026-07-08.
- **Workouts-page hero card + "Recent sessions" rename** — client/solo Workouts page shows an "Up next" hero (program name, phase/week, a Start button resolving the actual next scheduled session) when a program is assigned; "Session history" renamed to "Recent sessions," capped to last 5, date-only rows (both the client-facing page and the PT-facing client-profile Workouts tab). 8e9c26c, 2026-07-10.
- **`deleteProgram()` no longer destroys shared templates** — was deleting ANY template referenced by a program's phases with no ownership check; a shared/reusable standalone template slotted into a program would be destroyed the moment that program was deleted. Now only deletes templates it actually owns (its own `program_id`, or its own periodization-generated week clones via `generated_from_phase_id`). Found via Playwright test-flakiness investigation, not by inspection. 8e9c26c, 2026-07-10.
- **Workouts-page perf fix** — the flat templates query (up to 100 rows, nested exercise join) was always fetched even when a program was assigned and the result was never used; now only fetched when needed. Likely the same root cause as the still-open "app runs slow moving to workouts page" report from 2026-07-06 — needs Jake's live confirmation. 8e9c26c, 2026-07-10.
- **`client_programs` (+ 3 related tables) RLS fix, confirmed working end-to-end** — real (non-solo) clients previously couldn't see their assigned program at all (zero client-read policies on `client_programs`, `programs`, `program_phases`, `program_phase_workouts`). All 4 policies applied and confirmed via a fresh fixture test that queries the exact nested-embed shape the app itself uses. b126d5b, 2026-07-10.
- **Live crash fixed: empty phase on the Workouts page** — a phase with zero `program_phase_workouts` (a normal state, not a data error) crashed the accordion's `renderDays` on `undefined.forEach`; now shows an explicit empty-phase message. b79c152, 2026-07-10.
- Sudo/impersonation mode — "View as" button on client list (email-gated), amber banner, exit restores PT view
- Progress tabs — Body Weight and Personal Bests tabs have add buttons; Cardio explains auto-population
- Programs — create / edit / delete / phases / assign template to day / assign to client / edit start date / remove
- **`deleteProgram()` safety + cleanup** — blocks deletion with a toast (no confirm dialog, nothing touched) if any clients are assigned; otherwise deletes the program's own `workout_templates` before removing it, so deleting a program no longer leaves orphaned template debris behind. 2026-07-03. **2026-07-04:** a solo user's own self-assignment no longer counts as a blocking client (they can always delete their own program); the PT-facing toast now names the actual assigned clients instead of just a count.
- **Duplicate week** — phase card week header has a "Duplicate week" button (shown whenever the phase has an empty week ahead of its `duration_weeks`); copies that week's day/workout assignments into the next empty week as real, independent rows. Every week (1, duplicated, or periodization-generated) is now equally editable — add/remove/reassign a workout per day, not just week 1. Propagates to already-assigned clients (fresh per-client template clone per slot, same pattern as program assignment). 2026-07-04
- **Fork-on-edit for shared workout templates** — editing a workout (rename, add/edit/delete/reorder an exercise) through a program phase slot now clones the template first if it's still referenced by more than one `program_phase_workouts` row, so the edit applies only to that slot — the original stays untouched for any other slot/program still using it. Editing from the flat Workouts library list (not via a phase) is unaffected. 2026-07-04
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
- **Security/GDPR** — private storage buckets (now behaviourally verified, not just `public=false` — see storage-privacy.spec.js), PII-free logs, consent checkbox, data export, delete account (delete_current_user RPC confirmed to exist), **ICO breach procedure documented** (`breach-procedure.md`, 2026-07-12)
- **Behavioural RLS audit** (`tests/rls-audit.spec.js`, 2026-07-12) — the standing security regression gate. Probe A (a second coach who owns nothing reads 0 rows from all 22 tables), Probe B (a client sees only their own rows, with a positive lower bound so a denied query can't read as clean), Probe C (a client CAN read their assigned program through the full nested embed — the s23/s24 unexpected-DENY class), + a self-test that plants its own victim to prove the detector fires. Storage is a separate surface, covered by storage-privacy.spec.js. This replaced /deploy-check's old `qual='true'` grep, which caught none of the 4 real RLS gaps this project has had.
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

- **Full-codebase architecture audit — 2026-08-12, see `architecture-audit-2026-08-12.md`.** First-ever
  structured review of all 9 JS modules (prior review tooling only ever covered a diff or the top 2-3
  highest-churn files). 10 actionable findings filed as their own `bugs/*.md` rows (ownership-anchor gaps
  in app-progress.js/app-clients.js/app-programs.js/app-workouts.js, a 5th+ recurrence of the stored-XSS
  class across 5 files, mountModal() not adopted in 2 files, silent dashboard query-error swallowing, a
  stale goal-writes ledger item refiled with a new sibling, a chart-instance leak, dead code). Purely
  descriptive/architectural findings that aren't fixable bugs live only in the audit doc: `blueprint.md`
  is stale against the later solo-role redesign; `data-model.md` has 2 dangling `[[project-coachapp-
  architecture]]` wikilinks the audit doc is a candidate to satisfy; `dbq()`'s own definer (app-core.js)
  only uses it for 2 of its own 8 calls, which plausibly explains repo-wide low adoption (24 of 292
  `db.from()` calls); no in-repo README/architecture doc exists. No live RLS/SQL probe was run (out of
  scope) — every ownership-anchor finding is an app-level gap, not a confirmed exploit.
- **Runner set-accuracy build (session 11, 2026-07-03) — built + Playwright-verified, NOT YET PUSHED.** Per-set target display fixed (table mode was hardcoded to always show set 1's prescription — now dynamic, tracking the current working set like the wizard already did); delete-a-set added (wizard edit sheet + table rows, neither existed before); live rep tally (current exercise's running reps vs. same exercise's total last session, updates per set); swap/add exercise now opens the **literal same modal** used in the workout builder (library picker + 1RM group + full set-target builder + notes + superset) via a parameterised `showAddExerciseToTemplateModal(templateId, runnerCtx)` — corrected from an initial simplified picker after Jake's live-test feedback that "the same modal" meant literal reuse, not a stylistically-matching rebuild. Tick checkbox redesigned (white → green on complete, was near-invisible); delete button redesigned (red "Delete" label, was a bare ✕). All four came from Jake live-testing the approved build in the same session — see LOG for the full before/after per item.
- **Programs picker "Filter workouts below" search — ✅ Done 2026-07-11.** The recommended fix (replace the native `<select>` with a live-filtering tap-row list) shipped as `_openWorkoutPicker`. It closed **three** complaints at once: this one (a plain `<select>` gives no visible feedback until manually opened, so it looks broken), Jake's separate complaint that the assign-workout list would grow unmanageably, and his 2026-07-11 question about telling three same-named "Upper Body" workouts apart — all three were the same root cause, an `<option>` being unable to hold anything but plain text. app-programs v16.
- **`deleteProgram()` orphan-cleanup — ✅ Done (66bf1fd, 2026-07-03).** See "What's working" above. Historical backlog note: the fix stops *future* debris, but the existing ~993-template backlog on the main coach account (found while researching this fix) is separate and still needs its own cleanup pass — same class as the 65 orphans cleaned 2026-07-02. Jake's calls so far: skip building a one-click replace/update-assignment flow for now (just prevent silent double-assign); when calendar integration is built, it should write real per-session `calendar_events` rows, not a lighter marker.
- **Runner redesign → Hevy-style table logger** — ✅ v1 SHIPPED 2026-07-02 (6e6402a). See "What's working" above. **Phase 2 not started:** extend the table pattern (or an equivalent) to cardio/timed/unilateral/%1RM exercises, which still use the old one-set wizard. Two known gaps in v1, not yet resolved: (1) superset auto-switch (wizard jumps to the paired exercise on log; table never auto-advances by design, so a superset pair just sits there until the PT/client manually taps Next — only matters if real templates use `supersetGroup`, not yet checked against Jake's actual data); (2) bodyweight-in-table is code-reviewed but not live-verified (didn't hit a live bodyweight exercise in this session's test data).
- **Runner bug fixes** — ✅ Done (61d8bc7, 2026-07-02): set-counter cap, rest-timer-before-render (×3 branches), beep window 3s→5s, audio-unlock on both entry points. Playwright 48/48 + CI green + deployed. Audio-unlock real-device check deferred by Jake — mobile-web audio is a known limit a native app (Capacitor) will resolve properly.
- **Runner audit (vs the "Hevy" consumer-app bar)** — findings banked to roadmap.md as build items: (1) strength inputs aren't pre-filled → every set is a full retype vs Hevy's 1-tap repeat (biggest gap); (2) no plate calculator; (3) background rest-timer alerts need PWA/native; (4) last-session strip is strength-only. Rationale lives in the LLM wiki `coachapp-product-strategy` + `coachapp-client-app-benchmarks` pages.
- **Privacy policy page** — ✅ Done (v158, pushed 2026-06-29) — `/privacy-policy.html` live, consent link updated
- **Personal account (solo view)** — ✅ Done (v158–v162, pushed 2026-06-29) — PT | Personal view switcher, solo dashboard, programs self-assign, Workouts session accordion, RLS policies added
- **Performance logs RLS** — ✅ Done (2026-06-29) — 5 clean policies confirmed via pg_policies query
- **Invite email logo** — PT branding not yet included in invite email HTML (Edge Function not updated)
- **Progress-photos feature REMOVED 2026-07-12** (Jake, "for now"). Removed while fixing a **live cross-tenant leak** in its storage policies: `progress-photos` was `public=false` yet had 3 `storage.objects` policies scoped by `bucket_id` alone (`"Public read"` SELECT, `"Authenticated delete"` DELETE, `"Authenticated upload"` INSERT) — so any authenticated coach could read AND delete any client's photos. Reproduced live (a second coach downloaded a real 1.79MB photo, deleted another). Dropped all 3 (`scripts/fix-storage-rls-2026-07-12.sql`); the correctly path-scoped "Client/Coach manages …" policies already covered every legitimate op. Bucket + code retained (restorable from app-progress v9). See CRITICAL.md storage section + breach-procedure.md §6.
- **My Programs accordion Playwright tests** — conditional (skip if test client has no program). Test client may not have program assigned.
- **My Progress Strength tab** — PostgREST `!inner` join; not verified on live with real data
- **Program builder desktop layout** — cards may not span full width at wider viewports
- **Weekly check-in notification** — always shows Due if >7 days; no dismiss until submitted
- **Solo account Playwright tests** — ✅ 8 smoke tests added (v176); session detail slide-in smoke test added (v179); tests skip gracefully when E2E account has no solo client record

---

## Continuity block

### The consent gate's read MUST stay a separate, fail-open query (2026-08-24)
`_loadConsentState()` in `js/app-core.js` fetches `consented_at` / `consent_policy_version` on its own
and returns `null` on ANY error; `_needsConsent(null)` is `false`. This looks like an obvious tidy-up
— "why not just add those two columns to `loadUserInfo`'s select?" **Do not.** That select uses
`.single()`; if the columns are ever absent (a fresh environment, a rolled-back migration) it ERRORS,
which nulls `currentProfile`, which trips `showApp`'s fail-closed branch, which locks **every user out
of the whole app, including the owner**. A gate whose failure mode is total lockout is worse than the
gap it closes. `tests/consent-gate-2026-08-24.spec.js` pins the fail-open property directly; if you
find yourself deleting that test to make a refactor pass, the refactor is the bug.

Corollary: the gate is enforced inside `navigate()` rather than per-caller, because every route into
the app funnels through it — and `switchView()` still needs its OWN guard because it calls
`applyRoleUI()` before `navigate()`. Both bypasses (browser Back, "View as") were live and were found
by review, not by testing.

### A second helper doing the first one's job is where the bug will be (2026-08-22)
`_verifyOwnClientId` (app-clients) and `_verifyClientAccess` (app-core) both answer "may I write for
this clientId?". The duplicate is the one that broke "View as": `_verifyClientAccess` allows a coach
via `coach_id === me`, the strict-self copy does not. Before writing an ownership helper, grep for an
existing one — this codebase now has FOUR (`_verifyTemplateOwnership`, `_verifyClientAccess`,
`_verifyGoalAccess`, `_verifyMilestoneAccess`) and they must not multiply further.

### "View as" (sudo) is a coach-for-a-client render path, and it is easy to miss (2026-08-22)
`sudoAsClient` (app-dashboard.js:240) sets `window._sudoClientId` and forces
`currentProfile.role = 'client'` while `currentUser` stays the COACH. `renderClientDashboard` then
renders the weight / PB / check-in forms with the SUDO'D client's id. Any guard that assumes
"role === client means the id is my own record" breaks it. `_getCurrentClientId()` returns null in
sudo. Enumerating render sites by grepping the onclick is NOT enough — this one derives its id from a
window global set by a different function.

### An ownership guard's risk is refusing the LEGITIMATE user, not admitting a stranger (2026-08-22)
Every guard added on 2026-08-21/22 sits over RLS that already refuses. So a refusal test cannot go red
before the fix, and cannot detect the real failure either. **Every guard needs a happy-path test per
ROLE that can reach it** — coach, client, solo, and sudo. The sudo break shipped into a commit because
no sudo happy-path test existed.

### A check that cannot FAIL is worse than no check (2026-08-21)
`gates-fired` tested its patterns against the whole of LOG.md, so one occurrence in July kept it GREEN
forever. It was the only os-lint check that looks at behaviour rather than artifacts, and it was
structurally incapable of failing. Found only because Jake asked a question that forced someone to read
the source — not a repeatable discovery process. `os-lint --self-test` now points every check at a
fixture built to trip it and names any that stay green as DECORATIVE. **Never add an os-lint check
without an env override for its input** — an untestable check cannot be distinguished from a dead one.

### A green verdict over ZERO items is a switched-off checker (2026-08-21)
`loadSkills()` returns `[]` when the skills dir is missing; six checks then loop zero times and each
report success ("all **0** skills have name: + description:"). Rename `~/.claude/skills` and os-lint goes
almost entirely green while inspecting nothing. Anything reporting "all N ..." must assert N non-zero.

### The pre-push gate is 57 of 523 tests, and widening it is harder than it looks (2026-08-21)
Attempted and reverted. **A glob in playwright's positional args silently no-ops** — args are OR-ed filter
regexes, so `test a.spec.js b.spec.js missing-*.spec.js` exits 0 and just runs a+b. Verify any gate change
with `--list` and confirm the file COUNT, never the exit code. Selecting specs by filename prefix selects
an ERA, not a category. And the cross-tenant probes take their cleanup id from the offending session's own
`.insert().select()`, so they cannot clean up in exactly the regression case they detect — harden those
before any future widening.

### The assign modal closes BEFORE the clone work is awaited (2026-08-21)
`app-programs.js:758` removes `#apc-modal`; `:763` then awaits `_cloneProgramForClient`, which batches its
`client_program_workouts` inserts once at `:473`. **Modal-detach is not a barrier on the clone.** Any test
that treats it as one races the batch insert. The app is correct — this is a test-synchronisation trap.

### `escapeAttr` ROUND-TRIPS CLEANLY through a handler — the danger is the render site (2026-08-19)
Verified empirically, not reasoned: `escapeAttr(x)` inside `onclick="fn('${…}')"` survives the browser's
attribute un-escaping and JS's string-literal un-escaping, so the RUNTIME value is the ORIGINAL string —
no backslash. That makes the source call CORRECT, and means any ctx value it produced is raw
attacker-controlled text by the time something renders it.

`_ctx.backLabel` and `_ctx.clientName` were rendered RAW into innerHTML on that basis
(`app-workouts.js:1205`/`:1207`) — a live stored-XSS sink, 5th instance of the client→coach pattern
CRITICAL.md tracks. **Do not "fix" the source `escapeAttr` calls; escape at the RENDER site.**

`FREE_TEXT` in check-escaping.mjs matches `\.name\b`, which does NOT match `.clientName` — that naming is
why no checker saw it.

### A class guard is only closed once proven against EVERY syntax the codebase uses (2026-08-19)
The escapeAttr rule shipped 2026-08-16 matched only the INTERPOLATED shape, exited 0, and nine
CONCATENATED sites (`value="' + escapeAttr(x) + '"`, built inside `mini()`/`gmini()`) stayed live for
three days while the class was reported closed. **Grep how the construct is actually written** — the 14
characters before every call — rather than assuming a second form. A test now plants one of each shape.

**A rule that flags correct code is worse than no rule.** An indirection check (`const x = escapeAttr(…)`)
was written and removed within the hour when its only live hit turned out to be correct; the comment
explaining why is in the file so nobody re-adds it. Indirection is a full-file-review job, not a regex one.

### Reorder propagation permutes SLOTS, never positions (2026-08-19)
A sibling copy legitimately holds a different set of exercises, so copying `order_index` values across
would collide with, or displace, exercises the target has and the source does not.
`_propagateReorderToTemplates` permutes only the SHARED names into the source's relative order, **reusing
the slots they already occupy** — the set of order_index values in play never changes, so a collision is
impossible by construction rather than by care.

### A guard must sit ABOVE the code that dereferences state it does not need (2026-08-19)
`startIntervalPhaseTimer`'s zero-length refusal is placed before `stopIntervalTimer()`, which dereferences
`_runner`. `_runner` is `let`-declared in `app-workouts.js:3041`, so a top-level `let` in a classic script
creates NO window property and **no test can set it**. Refusing first is both better code and the only
reason the guard is testable without driving a whole workout.

### The runner labels per-side; the prescription formatter does too (2026-08-19)
`_fmtSetDetail` gained "per side" on 2026-08-14; `_buildTargetCols` did not, so the athlete entering L/R
had nothing saying the target was per side. Now carried on the reps LABEL ("8–10 REPS/SIDE") rather than a
fifth column — the target bar is horizontal on a 390px screen. AMRAP + unilateral keeps both as
"AMRAP/SIDE".


### A policy-refused write returns `{ data: [], error: null }` (2026-08-18)
No error, ZERO rows. **Checking only `error` reports a refusal as a success.** Every write that a user is
told succeeded must `.select()` and branch on ROWCOUNT — the pattern is `deletePerfLog`
(`js/app-progress.js`). Six sites were fixed in `ad83591` after `saveCoachNotes` was found showing a green
"Saved ✓" over a write that never landed (a client transferred between coaches keeps `coach_id` = the OLD
coach on every pre-transfer `workout_logs` row).

**The asymmetry is deliberate and must not be "fixed":** in `_propagateExerciseChangeToTemplates`, only the
INSERT branch counts 0 rows as a failure. For update/delete, 0 rows is the documented "this target doesn't
contain that exercise" no-op, and counting it would fire alarms on the common case. There is a test pinning
this (`tests/silent-refusal-2026-08-18.spec.js`).

### A recovery link ESTABLISHES A SESSION (2026-08-18)
So `currentUser` is set the moment the user opens it. Without a guard, `showApp()` fires and the app boots
straight past the set-password form into the dashboard — the user lands logged in with the single-use link
already burned and no password set. `onAuthStateChange` must show the form on `PASSWORD_RECOVERY` **and** on
a `type=recovery` hash, and the app shell must stay hidden until the password is set
(`js/app-progress.js`, tests in `tests/password-reset-2026-08-18.spec.js`). This used to be a bare
`if (event === 'PASSWORD_RECOVERY') return`, which is why a dashboard-sent recovery link did nothing at all.

`_initialHash` is captured at the TOP of `app-core.js`, before the supabase client is constructed, because
supabase-js consumes and clears the hash while establishing the session. Anything reading the link type must
read `_initialHash`, never `window.location.hash`.

### `_cleanTemplateSets` is an ALLOWLIST, and EVERY writer must call it (2026-08-18)
42 emitted keys; a key it stops emitting is silently gone from every saved row with no error at any layer.
`tests/stale-set-fields-2026-08-18.spec.js` pins the count — if it changes, a field was added or removed on
the save path for every template exercise.

`flushTemplateSets` deliberately PRESERVES fields the current type does not render, so the metric-type gates
inside `_cleanTemplateSets` are the only thing stopping a stale field riding onto the wrong exercise type.
**The EDIT path skipped it entirely until `53071cf`** while both siblings cleaned.

Gate **per key, never per family**: `targetHeightCm` only on `jump_height`, `targetDistanceM` only on
`jump_distance`. A family-wide gate still let a `jump_distance` row keep a height, and `_fmtSetDetail`
prefers height unconditionally — so the preview showed "40cm" where the runner showed "2.4 m".

### `_fmtSetDetail` infers the exercise type from SET DATA, the runner from `metric_type` (2026-08-18)
`} else if (s.targetHeightCm || s.targetDistanceM) {` (`js/app-workouts.js:290`). That is why a stale field
makes the coach's plan preview and the athlete's in-gym screen disagree about what the exercise is, across
six surfaces. **Fix stale data at the gate, never in the renderer** — patching `_fmtSetDetail` leaves the
bad data on disk and hides only one of the six.

### Playwright `has-text()` is a case-insensitive SUBSTRING match (2026-08-18)
`button:has-text("End")` matched a new **"S-end- reset link"** button and broke 24 runner assertions. Use
`button:text-is("End")` for an exact match. 30 locators across 5 files were tightened; adding any button
whose label contains a shorter button's label will otherwise break unrelated specs.

### A block ends where the same programme's NEXT RUN begins (2026-08-17)
`client_program_blocks.ended_at` is the day RESTART WAS PRESSED, not the day the block's training finished,
and the next block's start is snapped back to ITS OWN Monday — so the two can overlap by up to six days or
leave a gap. **No rule about the end date alone can fix that.** `_ptsInBlock` is a plainly INCLUSIVE filter;
the overlap is resolved once in `_clampBlockChain` (`js/app-progress.js`), scoped to the same `progKey` so
two genuinely concurrent programmes still overlap legitimately.

### The BW toggle must render unconditionally (2026-08-17)
It used to render only `${s.bodyweight ? … : ''}` — and that toggle is the ONLY thing that can set the flag.
A bootstrap deadlock: no template exercise created after that gate could ever be marked bodyweight, which
also made the bodyweight reps-charting unreachable for new data. Same defect that got `assisted` deleted on
2026-08-11. `_BW_TYPES` is deliberately WIDER than `_AMRAP_TYPES` (it includes `timed_hold` — the runner has
always rendered a BW cell for a hold).

### NEVER use `toISOString()` for a calendar date (2026-08-16)
It converts to UTC, so a local-midnight Date in any UTC+ zone (Europe/London under BST — most of the year)
reports the PREVIOUS day. Use `_ymdLocal` (`js/app-core.js`). This silently turned `_mondayOfWeek`'s Monday
into a Sunday. Millisecond date arithmetic is equally unsafe across a DST change — local midnight to local
midnight over a spring-forward is 7 days MINUS an hour; anchor at NOON (`_blockWeekIndex`).


### A weight unit is a TEST DIMENSION, not a display detail (2026-08-14)
`weightToPref` (`js/app-core.js:85`) returns a **number** in kg but a **STRING** in lb — its lb branch exits
through `_stripTrailingZero`, which is `String(v).replace(...)`. So **never call a number method on its
return value**; `parseFloat` first, exactly as `fmtWeight` (`js/app-core.js:110`) does. Writing
`weightToPref(x).toFixed(1)` threw `TypeError` and killed the entire Progress page for any lb user with a
recorded 1RM — caught by pre-push review, never shipped.

**The whole Playwright suite runs at the `kg` default (`js/app-core.js:72`) and nothing flips it**, so this
class is invisible to every existing test. Any new surface rendering a weight needs a kg/lb matrix test —
the precedent is `tests/screenshot-feedback-2026-08-14.spec.js` ("1RM grid survives a non-default weight
unit"). Same reasoning applies to `jumpHeightToPref` / `cardioDistanceToPref`, which have the identical
number-or-string shape.

### `flushTemplateSets` preserves un-rendered fields — so every per-set flag needs a type gate (2026-08-14)
Switching an exercise's Type does **not** clear set fields the new type doesn't render; that is deliberate
(it lets a user flip back without retyping). The consequence: a per-set flag toggled under one type survives
into another where it is meaningless. `amrap` toggled on weight_reps then switched to Jump produced "AMRAP
jumps" in the runner, "AMRAP:" as the set label in two detail views, and **nothing** in the collapsed day
row — three surfaces disagreeing about one set.

**Gate it once at `_cleanTemplateSets`, never at the render sites** — that function takes `metricType` as a
third argument for exactly this (`_AMRAP_TYPES`). The same class already has a guard for stale `weight` on a
jump set in `_buildTargetCols` (`js/app-runner.js`). Note `_cleanTemplateSets` is an **allowlist**, so a
caller that forgets the third argument silently drops the flag — that is a feature, and a test caught it.

### Cardio/interval — one metric_type now, not two (2026-08-09)
`'cardio'` is **no longer a selectable metric_type** — the builder's exercise-type `<select>` only offers
`'interval'` for the whole cardio family now (Jake: "having cardio and intervals as 2 separate categories
is redundant"). A steady, single-round effort is just an interval block with `sets:1, cycles:1` and zero
warmup/rest/recovery/cooldown — **derive "is this steady" from the block's actual fields, never store a
separate flag** (`_isSteadyIntervalBlock(b)` in app-workouts.js is the one shared predicate; the runner's
`_isSteadyEffortBlock(ex)` delegates to it — don't reimplement this check a third time). The runner's
manual/no-timer LOG button now works for a steady-effort block too (previously interval-only-off); a
genuine multi-round/multi-cycle block still requires the live timer. `'cardio'` stays DB-allowed (no CHECK
constraint tightened) since historical `workout_log_exercises` rows and the `'amrap'` precedent both rely
on the DB permitting values the UI no longer writes — don't "clean up" that constraint without checking
history first. All real `workout_template_exercises`/`exercises` rows were migrated live
(`scripts/merge-cardio-into-interval-2026-08-09.sql`, backed up first) — if you ever see a `metric_type`
value of `'cardio'` in live data again, that's new/unexpected, not a leftover.

### New-account provisioning — Edge Function pattern, not raw service-key scripts (2026-08-09)
Public self-signup is gone for good (2026-07-24 incident) and there is still no self-serve solo-account
signup (deliberately deferred, Jake's own call). The supported way to create a brand-new account (coach,
client, or now solo) that needs privileged (service-role / bypasses-RLS) logic is a **Supabase Edge
Function**, deployed via the Dashboard's browser-based "Via Editor" flow (no CLI is installed or tracked in
this repo — don't assume one exists). `supabase/functions/invite-solo-user/index.ts` is the second such
function (after the pre-existing, undocumented-source `invite-client`) and the first one with its source
actually tracked in this repo — do that for any future Edge Function too, even though Supabase itself
doesn't require it. **Every Edge Function that does anything privileged must independently re-verify the
caller's identity server-side** (`supabase.auth.getUser()` against the caller's own Authorization header,
never a body param) — a client-side email/role gate hiding a button is a UI convenience only, not a
security boundary; this was verified live (a real non-owner session got a genuine 403 from the deployed
function, not just a hidden button). The one-off local script pattern (`SUPABASE_SERVICE_KEY=... node
scripts/*.cjs`, service-role key pasted into `.env` each time) still exists as a documented fallback
(`scripts/invite-solo-user.cjs`) but is explicitly NOT how Jake wants repeated/at-scale operations done —
he pushed back hard on it ("too convoluted... won't scale"). Default to the Edge Function + in-app UI
shape for anything a coach/Jake will need to do more than once.

### Week-tabs display model (2026-07-18) — Programs builder + client Workouts page
Both surfaces render phase→week→day→workout the same way: **weeks = tabs** (`.week-tab`, one week on screen at a time), **days = rows**, and a day/slot **opens its workout inline**. The full-screen session-detail slider (`openSessionDetail`) is **no longer called** by either surface — it survives only for the standalone Templates list (app-workouts.js). Read page (`renderClientWorkoutsPage`) pre-renders every week into hidden `.rw-week` panels toggled by `_selectReadWeek`; the first phase (`pi===0`) is open by default. Builder (`loadAllPhaseWorkouts`) renders per-phase week tabs + **only the active week's grid**, cached in `window._builderWeekData[phaseId]` and re-rendered by `_selectBuilderWeek` (no refetch); the active week persists in `window._builderActiveWeek[phaseId]`, clamped to a valid week on every render. Builder slots expand inline (`_toggleBuilderSlot`) to **Edit** (`_editPhaseWorkout` → `openTemplate` with the program back-ctx — same ctx the removed slider passed) / **Remove** / **Save to Library** (`saveTemplateToLibrary(id, btnEl)`, moved onto the slot from the removed drawer). Responsive: `.pwk-days` is a vertical list on mobile, a grid ≥768px. **Any name/exercise-name interpolated into these renders must be `escapeHtml`'d** — the DAY-header `sessionSummary` was a raw coach→client XSS, escaped 2026-07-18.

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

**Genuinely-solo accounts are a separate mechanism (2026-08-01) — `role='solo'` is now NATIVE, not cycled.**
`clients.user_id` has a real UNIQUE constraint (`clients_user_id_idx`, documented `tests/gdpr-export.spec.js:84-97`)
— an account can hold AT MOST ONE `clients` row, ever, so a real account is either a master account (matches
one of the two `Promise.all` queries above) OR a genuinely-solo account, never a hybrid of both. For the
latter, `profiles.role` is stored as `'solo'` directly in the DB (migrated by `scripts/migrate-solo-role-
2026-08-01.sql`, one real account today) — `loadUserInfo()` treats `role==='solo'` (or the pre-migration
transitional shape `role==='coach' && solo_only===true`, kept as a permanent OR-clause defense against
deploy-before-migration ordering, not a temporary shim) as its own branch: looks up the self-referential
`clients` row for `_soloClientId`, never touches `_masterAccount`, and `switchView()` is therefore always a
no-op for this account (it never had a coach view to switch to). `solo_only` stays in the schema, unused as
the steady-state signal but doubling as that transitional safety net.

### Storage — signed URLs
Private buckets: logos (604800s = 7 days), progress-photos (3600s = 1hr). Never `getPublicUrl`. Use `createSignedUrl` (single) or `createSignedUrls` (batch).

### Cache busting
Each of the 9 module files (incl. `starter-content.js`, shipped 2026-07-12 — don't glob for `app-*.js`, enumerate
`js/*.js` on disk) has its own independent `?v=N`. Any commit that changes a module file must bump that file's own
version in index.html — bump only the files that changed, not all 9. Current (2026-08-01, 2nd save, local-only —
see Live State at top for push status): app-core v=8 · app-dashboard v=8 · app-clients v=9 · app-programs v=27 ·
app-calendar-goals v=7 · app-workouts v=48 · app-runner v=48 · app-progress v=31 · starter-content v=5.

### metric_type — the single source of exercise shape (shipped live 2026-07-19, 95e8e8f)
`metric_type` is now first-class and intrinsic to an exercise: one of `weight_reps` · `unilateral` · `timed_hold` · `cardio` · `jump_height` · `jump_distance` (6 values, CHECK-constrained). It lives on `exercises` (source-of-truth default), denormalized onto `workout_template_exercises` + `workout_log_exercises` (same flow as legacy `exercise_type`, because a logged row's `exercise_id` is nullable so history can't join back). **AMRAP is NOT a metric_type — it's a per-set flag** (like `bodyweight`/`assisted`); it tracks reps, same as `weight_reps`. The builder picker sets metric_type and DERIVES legacy `exercise_type` (cardio/strength) + per-set `unilateral`/`timed` flags from it (`_deriveFromMetricType`, app-workouts.js) so the current runner keeps working. The runner routes by metric_type (`_isPlainStrengthExercise` → `_METRIC_TABLE_TYPES` via `_exMetricType`, with a legacy-flag fallback): the fast table handles all 5 strength types, only cardio stays on the wizard. `workout_log_sets` carries `avg_hr`/`max_hr`/`height_cm`/`side` (populated by ②b/②c/②d).

### Strength table (Hevy-style runner v1)
`_isPlainStrengthExercise(ex)` gates the table vs. wizard: excludes cardio, any set with `timed`/`unilateral`/`intensityMin`, and carry/sled/lunge-name exercises (regex-matched, same list used elsewhere). `ex.tableRows` = `[{weight, reps, done}]`, lazily built by `_ensureTableRows` (seeded from `_prevSetsByIndex`, keyed by `set_number - 1` against `_runner.lastSession[ex.name]`). `toggleTableSet(i)` flips `done` and calls `_syncLoggedSetsFromTable`, which **rebuilds** `ex.loggedSets` from scratch each time (`filter(done).map({weight,reps})`) rather than pushing — verified safe: every consumer (`saveRunnerSession`, `showRunnerFinish`, header dots) only reads `.length`/iterates, never holds a stale reference. `renderRunnerLastSession` was extended (not just the table added) to retroactively backfill blank rows if the async last-session fetch resolves after the table's first paint — guarded against re-triggering itself (row already non-blank → skip → no re-render loop).

### Deleting a program's templates — ownership AND live-references, always both (2026-07-11)
Any code path that deletes `workout_templates` belonging to a program/phase must ask **two separate questions**, and they are NOT the same question:
1. **Ownership** — is this template the program's own (`program_id` match) or its own periodization week-clone (`generated_from_phase_id` match)? A coach's reusable **standalone** template merely slotted into a week is *not ours to delete* — destroying it rips it out of every other program too (the `deleteProgram()` data-loss bug, 2026-07-10).
2. **Live references** — "Duplicate week" is *cheap by design*: the new week's rows share the **source week's `template_id`** until someone forks on edit (`_resolveEditableTemplateId`). So a template we genuinely own may still be needed by a surviving row in another week.

Both live in **`_deleteOwnedUnreferencedTemplates(templateIds, programId, phaseId)`**, shared by `deletePhaseWeek` and `_cleanupPhaseWeeksBeyond`. They are in one helper *precisely because they silently diverged before*: `deletePhaseWeek` got the guards on 2026-07-10, `_cleanupPhaseWeeksBeyond` did not — and that gap destroyed real Week-1 workouts until 2026-07-11. **Never add a third delete path without calling this helper.** Callers must delete their own `program_phase_workouts` rows FIRST, so whatever still references the template afterwards is a genuine survivor.

### `is_personal` — a UI display split, NOT a security boundary (2026-07-11)
`exercises.is_personal` (2026-07-10) and `workout_templates.is_personal` (2026-07-11) exist because the PT account and the solo/Personal account are **the same login** — same `auth.uid()`, same `coach_id`. Without the flag, anything built in Personal view bleeds into the PT's real client-facing library and vice versa. Every standalone-template/exercise read filters `.eq('is_personal', currentProfile?.role === 'solo')`; `saveNewTemplate`/`saveNewExercise`/`_createExerciseFromPicker` stamp it from the same expression. Clone paths (`_cloneSharedMasterTemplate`, `_cloneTemplateForClient`, `generatePhasePeriodization`) **carry the source's value over** rather than recomputing from the current role — and each of their source `select()`s must therefore include `is_personal` explicitly (a plain object-literal insert silently drops an `undefined`, falling back to the DB default; this exact no-op was caught by the multi-agent review on 2026-07-11).

**Deliberately NOT enforced at RLS — do not "fix" this.** The client Dashboard hero (`app-dashboard.js`) and client Calendar (`app-calendar-goals.js`) embed the **master** templates through `program_phase_workouts` — not the client's clones. A program built in Personal view carries `is_personal = true` masters, so an RLS `is_personal = false` restriction would silently null that embed and break the client's dashboard/calendar (the exact PostgREST silent-null failure mode from sessions 23/24). `is_personal` answers "which of Jake's two libraries does this belong to," not "who may read this." The genuine multi-tenant boundary is `coach_id` + `client_id`, and that is what the RLS fix of 2026-07-11 tightened.

### Periodization — week_number / tier
`program_phases.periodization_type` (`'linear'`/`'undulating'`/null) + `periodization_config` (jsonb). `program_phase_workouts.week_number` (default 1) lets one phase hold distinct day/template assignments per week — phases that never use periodization just have week_number=1 rows, which the client calendar/workouts-page render as repeating every week (legacy behaviour, unchanged). `program_phase_workouts.tier` (`'heavy'`/`'moderate'`/`'light'`) is undulating-only, set per day-slot. `generatePhasePeriodization(phaseId, programId)` clones Week 1's templates into weeks 2..duration_weeks, recalculating only sets where `intensityMin`/`intensityMax` is set — everything else (reps, rest, tempo) is copied unchanged. Regeneration is idempotent (`_cleanupPhaseWeeksBeyond` deletes stale weeks + their templates, both master and any already-propagated client copies, before regenerating) — the same helper runs when a phase's `duration_weeks` is edited down. Client propagation: assigning a program clones week_number through (`_cloneTemplateForClient`/`_cloneProgramForClient`); if a client is *already* assigned when new weeks are generated, `generatePhasePeriodization` propagates to them too.

**%1RM vs RPE — not interchangeable (clarified 2026-07-03):** the phase-level Start/End % (or undulating tiers) only scales sets that were built with the **%1RM field** in the set editor (`intensityMin`/`intensityMax` on that set). Sets built with RPE/RIR instead are left completely untouched by design — periodization has nothing to scale on an RPE-based set, so the session-detail drawer correctly shows nothing extra for them. This is a per-exercise, per-set authoring choice in the set editor, not a phase-wide toggle. Found via Jake's test program: Phase 1 set to Linear 70→80%, but Bench/Squat/Row were all built with RPE, so nothing visibly changed — expected behaviour, not a bug.

### Unit preference — `window._unitPrefs`, canonical storage, conversion only at the boundary (2026-07-25)
`profiles.weight_unit`/`jump_height_unit`/`cardio_distance_unit` (kg/lb, cm/in, km/mi — independent, not one
metric/imperial switch) are read into `window._unitPrefs` by `loadUserInfo()` and used by every render/save
function that touches a weight, jump height, or cardio distance. **Storage is always canonical (kg/cm/metres) —
never write a converted value to the DB.** Conversion happens ONLY through the shared helpers: `weightToPref`/
`weightFromPref`/`fmtWeight` and `jumpHeightToPref`/`jumpHeightFromPref`/`fmtJumpHeight` (app-core.js),
`distanceToPref`/`distanceFromPref` extending `fmtDistanceM` (app-workouts.js). `*FromPref` always returns a
NUMBER (via `parseFloat`), even for the native/unconverted unit — a raw `.value` string read straight off an
input is NOT the same type; don't assume string-vs-string equality still holds after routing a field through
one of these. `*ToPref`'s native-unit branch is a bare passthrough with **no forced rounding** — a display site
that always showed a fixed decimal count (e.g. `.toFixed(1)`) before the toggle existed must pass `fmtWeight`'s
`decimals` option explicitly, or it will silently lose that formatting for kg-only users (this exact regression
shipped and was caught by review in the same session that added the option). Entry-field reads must use the
explicit existence-check pattern — `const el = document.getElementById(id); s.field = el ? (converter(el.value)
?? fallback) : s.field` — not a naive `?.value ?? s.field` wrapped in a converter, which can't tell "field not
rendered for this metric type" from "user deliberately cleared it." **Account-wide, not per-role**: coach and
solo share one `profiles` row by design (same `auth.uid()`) so they always share one unit preference; a real,
separate client account has its own row and is never affected by their coach's setting — do not "fix" either
of those.

**Second entry point, 2026-08-08**: `_saveUnitPrefs(weight, jumpHeight, cardioDistance)` (app-core.js) is now the
shared DB-write core — `saveSettingsUnits()` (Settings page) and the new `_saveQuickPrefs()` (the quick-prefs
popover, reachable from the runner and builder via `_quickPrefsIconHtml()`/`_openQuickPrefsPopover()`) both
call it rather than duplicating the update/overwrite-`window._unitPrefs` logic. Same popover also holds the
4 cardio-capture-metric toggles (`_cardioCaptureToggles`, localStorage — see the cardio capture ledger row,
2026-08-08) — one shared preference surface, two entry points, not two divergent copies.

### Playwright nav clicks — always clickVisible/waitForVisible, never raw page.click (2026-07-25)
`playwright.config.js` now genuinely runs at 390×844 (a config bug previously made every test run at desktop's
1280×720 instead — fixed). The app's `renderNav()` (`js/app-core.js`) intentionally writes an identical
`data-page="x"` link into BOTH the sidebar nav and the bottom nav for every page — that's correct, normal
product behavior, not a bug to "fix" — CSS alone (a 900px breakpoint) decides which is shown. Any NEW Playwright
test that clicks a nav item, the Personal view-switcher, or signs out must use `clickVisible`/`waitForVisible`
(`tests/helpers.js`) — e.g. `clickVisible(page, '[data-page="workouts"]')` or
`clickVisible(page, ['#vs-personal', '#mvs-personal'])` — never a bare `page.click('[data-page="x"]')` or
`page.click('text=Personal')`, which will resolve to the sidebar's (hidden, at 390px) copy and time out. A
locator that specifically needs "the currently-visible mobile copy" (e.g. an existence assertion, not a click)
can target `.bottom-nav-item[data-page="x"]` directly, since the suite's only project is fixed at real mobile
width.

### 1RM assignment-time check
`_getProgramOneRMStatus(programId, clientId)` scans Week 1 only (sufficient — generated weeks reuse the same exercise names) for %1RM-tagged sets, diffs against `client_1rms` (trim+lowercase exact match — same limitation as the rest of the app; exercise_id-based matching is a deferred future decision, see roadmap). `_renderOneRMQuickEntry(idPrefix, name)` is the shared toggleable kg/Epley-estimate row component — parameterized by idPrefix so multiple rows coexist. `window._missingOneRMExercises` holds the current checklist's missing-name list (index-matched to `mor-N` DOM ids) between refresh and save. `_refreshMissingOneRMs` is token-guarded (`_oneRMRefreshToken`) so a stale in-flight refresh can't overwrite a newer one if the PT changes the client/program selection quickly. Wired into both assign entry points (`showAssignProgramModal`/`showAssignProgramToClientModal`) — 1RM entries must be saved *before* the modal is removed, not after (DOM reads fail silently on a detached modal).

---

## Open to-dos for Jake

**This table is the BUG LEDGER. It is the system's only organ for work owed.**

_Intake rule (2026-07-13): the moment Jake reports a bug, it becomes a row here — **before** investigation
starts, not at `/save`. Every row carries a `Reported` date and a `Status`._

**Statuses:** `open` (mine to fix) · `fixed — awaiting Jake` (shipped, needs his eyes) · `confirmed` (done, verified) · `deferred (Jake)` (he explicitly chose to park it — only he may set this).

> ### 🔒 CLOSURE RULE — read before removing any row
> A Jake-reported item may be closed **only** by:
> **(a)** Jake confirming it, or
> **(b)** a test that went **RED before the fix and GREEN after**.
>
> Never by inference. Never by "likely the same root cause." Never because Playwright covers an adjacent
> flow. Never because a robot looked instead of Jake.
>
> This rule exists because the "slow Workouts page" report was closed on a guess on 2026-07-06, stayed
> broken, and Jake re-reported it seven days later. **An empty to-do list is not a good outcome — an
> honest one is.** `os-lint` turns any `open` row older than 7 days RED at session start.

**The bug ledger now lives in [`bugs/`](bugs/) — one file per bug, not a table.**

It was 125 rows in a markdown table right here, and that shape caused two silent failures of its own:

- Prose containing an unescaped `|` split a row into the wrong cells, so description text landed in
  the Status column and `os-lint` skipped the row entirely. Six rows were in that state.
- The Status cell sat at the far right of lines up to **6,923 characters**, so it went unedited when
  a fix was written onto the front of the row. Six rows said FIXED in their text and `open` in their
  cell — the oldest for 17 days, inflating the RED count until the genuinely-open rows were buried.

Both are structural properties of using a table cell as a database field, so the table is gone.

Each bug is `bugs/NNN-slug.md` with YAML frontmatter:

```yaml
---
id: bug-001
status: open | fixed-awaiting-jake | confirmed | deferred | closed
priority: critical | high | medium | low | unset
reported: 2026-08-09
status_detail: "the original free-text status, when it said more than the enum"
---
```

`status` is a five-value enum a machine can read. Anything extra a human wrote is preserved verbatim
in `status_detail`, and the full original prose is the body of the file. **One machine-readable fact,
one human note — never one field trying to be both.** That split is the whole point.

**Counts are deliberately NOT repeated here.** `os-lint` reads `bugs/` directly at session start and
reports stale-bug and drift counts from the files themselves. A count written into this file would
recreate exactly the two-fields-one-fact drift the move exists to end.

`os-lint` also runs a `bug-files` check: any file whose frontmatter will not parse is RED, because a
malformed bug file is invisible to the other checks in precisely the way a pipe-split row used to be.

**The closure rule is unchanged.** A Jake-reported item closes ONLY on (a) Jake confirming it, or
(b) a test that went red before the fix and green after. Never by inference, never because a commit
message claimed it. Set `status: fixed-awaiting-jake` when a fix ships — that is a relabel, not a close.

*Converted 2026-08-11 from the STATUS.md table; 125 rows in, 125 files out, round-trip verified.*


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

## Decisions

### Is the runner a plagiarism risk? (Jake, 2026-07-13) — **No. Low risk, no change needed.**

**The honest answer: what you copied isn't the kind of thing that's protected.**

A set-by-set table with SET / KG / REPS / ✓ columns is a *functional layout* for logging weight training,
and functional layouts are not protectable — the same grid is used by Hevy, Strong, Jefit, Progression and
half the spreadsheets in every gym. Copyright protects the *expression* (their exact icons, illustrations,
colour system, wording, logo, sound design), not the *idea* of a checkable set-row, and not the arrangement
that any app logging the same data would converge on. Patents would be the other risk, and a UI arrangement
like this is very unlikely to hold one. There's no meaningful case here.

**Three things to actually do, all cheap:**
1. **Stop calling it "Hevy-style" in anything user-facing.** It's fine as internal shorthand (STATUS, LOG,
   commits — that's where it lives today, which is fine). It is *not* fine in marketing copy, App Store text,
   or a beta email. Naming a competitor as your descriptor is the one thing that turns "convergent design"
   into "he admits he copied it," and it's the only real self-inflicted risk in this whole question.
2. **Never lift their assets** — icons, illustrations, exact microcopy, colour tokens, the rest-timer sound.
   Nothing in the codebase does today.
3. **Note that you've already diverged where it matters.** You *removed* two of the features you'd imported
   from the Hevy benchmark (pre-fill/1-tap-repeat and the plate calculator) because real gym use rejected
   them, and you replaced the PREVIOUS column with ghost text — a genuinely different solution. The runner
   is converging on *your* training, not theirs. That's both the better product argument and, incidentally,
   the better legal one.

**Not legal advice** — if CoachApp ever raises money or gets acquired, a real IP lawyer does a real review.
For a beta with a handful of PTs, this is not a risk worth spending a session on.

---

## Build history snapshot — ARCHIVE, pre-2026-06-25 only

> **This is not current state. For live version numbers read `## Live state` at the top of this file;
> for session history read `LOG.md`.** Until OS v3 this table also carried twelve verbose
> `Pushed 2026-07-0x` rows whose version numbers (`app-programs v=11`, `app-workouts v=16`,
> `app-runner v=16`) had been frozen since 2026-07-08 and contradicted `## Live state` in this same
> file by 45 days. Every one of those sessions has a `## <date>` entry in `LOG.md`, so they were
> deleted 2026-08-23. The rows below are kept because they predate LOG.md (earliest entry
> 2026-06-25) and exist nowhere else.

| Version | What shipped |
|---|---|
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
