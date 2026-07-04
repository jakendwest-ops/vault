# CoachApp — STATUS
_Last updated: 2026-07-04 (session 15)_

---

## Live state

**App version:** app-core/dashboard v=1 · app-clients v=2 · app-programs v=9 · app-calendar-goals v=2 · app-workouts v=11 · app-runner v=10 · app-progress v=3
**Hosting:** GitHub Pages — https://jakendwest-ops.github.io/coachapp — deploy source switched 2026-07-03 from legacy branch-deploy to Actions-only (`build_type: workflow`); see CRITICAL.md timeline for why
**CSS version:** v=3 (main.css)
**Last push:** f997474 — Scoped 5 `clients` queries in app-clients.js (openClient, renderClientOverview, saveUpdateEmail, showEditClientModal, saveEditClient) by `coach_id` in addition to `id` — defense-in-depth, RLS already blocked cross-tenant access; 2-agent review confirmed all 5 are PT-only code paths so no client/solo flow could break. Also tightened the `client-workout.spec.js` "session history" locator to be onclick-scoped like its siblings. Along the way, corrected a stale fact in roadmap.md: solo accounts' own client record has `coach_id = NULL`, not `coach_id = auth.uid()` as previously documented — verified against `app-core.js:132`. (Previous: 84f9267 — Runner exercise-picker freeze fix + silent rest-timer beep fix. Root cause: Swap/Add exercise shared one hardcoded modal id (`add-to-template-modal`); a fast double-tap during the picker's own fetch latency opened two overlays, and `getElementById`/`closeModal` only ever operate on the first — the visible modal could never be closed. Found live during Jake's real gym session 2026-07-04 (personal account) — forced a reload that lost the entire in-progress workout (confirmed no partial/orphaned DB rows resulted, since `saveRunnerSession` was never reached). Fixed with a pending-flag guard + button-disable while the fetch is in flight + error handling that didn't exist before. Also replaced the 5-second tone-beep countdown with spoken numbers across all three timers (rest/timed-set/cardio interval) — hypothesis: iOS suspends the Web Audio API on screen-lock/backgrounding mid-rest far more readily than the Web Speech API the working "10 seconds" voice cue already uses. 3-agent review caught a real bug in the fix itself (cardio interval timer never primed speech synthesis on its own gesture — would have made its new spoken countdown silently never fire); fixed same session. New Playwright regression test reproduces the exact double-tap race. 60/60 Playwright green (some run-to-run flakiness observed, confirmed environmental/network-timing, not code-related — a different unrelated test flakes each run and passes on retry). (Previous: 730738a — Duplicate week (phase card "Duplicate week" button copies a week's day/workout assignments into the next empty week as real independent `program_phase_workouts` rows; every week — 1, manually duplicated, or periodization-generated — is now equally editable via one unified `renderPhaseWeekGrid`, replacing the old week-1-only editable grid); fork-on-edit for shared workout templates (`_resolveEditableTemplateId`/`_cloneSharedMasterTemplate` in app-workouts.js — editing a template through a phase slot now clones it first if that template is referenced by more than one `program_phase_workouts` row, so a rename/edit via one slot never silently mutates a template reused elsewhere, e.g. the same template assigned to both Monday and Tuesday); `deleteProgram()` fix (a solo user's own self-assignment no longer counts as a blocking client; the PT-facing block toast now names the actual assigned clients instead of just a count). 3-agent review (security/scoping, solo-mode, duplicates/render-safety) found one real issue — the "Generate weeks" confirm dialog's wording only mentioned "previously generated" weeks, stale now that weeks 2+ can hold manual content too — fixed same session. 3 new Playwright tests added (59/59 green). Live-verified in preview against real Supabase data (not just Playwright) before push. (Previous: ec30ebf — trivial empty verification commit confirming coachapp pushes still work after the PAT security cleanup; no app code. 609d5ee — removed dead graphify build-tooling; no app code.)
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
- **Strength table (Hevy-style)** — plain-strength exercises show an all-sets-visible table (SET/PREVIOUS/KG/REPS/✓) instead of the one-set wizard; previous-session values pre-fill KG/REPS for 1-tap repeat; tap ✓ to complete a set (fires rest timer, non-blocking — table stays visible/editable underneath); uncheck to undo; "+ Add set" appends a row; free-edit any row any time, no forced order. Cardio/timed/unilateral/%1RM exercises stay on the existing wizard (phase 2). v1, shipped 2026-07-02 (6e6402a). **2026-07-03 (v7):** target-info bar restored above the table (reps/RPE-RIR/tempo/rest, or %1RM→kg target when a 1RM is known — `_buildTargetCols`/`_renderTargetBarHtml`); rest timer now renders **inline** in that section as a plain 0:00 countdown instead of a fixed bar covering the runner header; unchecked ✓ button now has a visible outline
- **Mid-workout swap / add exercise (session-only)** — runner header has "⇄ Swap exercise" and "+ Add exercise" (`showExercisePicker`); swap replaces the current exercise in-memory and refetches its previous-session data under the new name; add appends a new exercise and jumps to it. Neither writes to `workout_templates` — today's log only. Picker modal forced to z-index 1000 to sit above the runner's fullscreen layer. v7, 2026-07-03
- **Voice cue fixed** — `_unlockSpeech()` now does a real gesture-tied `speak()` to prime the engine (was only calling `cancel()`, which never registered the gesture, so the "10 seconds" cue silently never fired); `speakCue()` selects a female voice. v7, 2026-07-03
- **Create-new-workout auto-assigns to its day slot** — building a template via the phase grid's "＋ Create new workout" now inserts the `program_phase_workouts` row back into the originating slot instead of leaving it empty. app-programs v6, 2026-07-03
- **Edit from the session-detail drawer** — the read-only session preview now has an Edit button (hidden for `client` role) that hands off to the full template editor with context, so the propagate-to-all-sessions prompt still fires. app-workouts v8, 2026-07-03
- **Phase card shows the periodization range** — e.g. "Linear (70→80%)" / undulating tier %s, instead of just the type name. app-programs v6, 2026-07-03
- **Runner exercise-picker freeze fixed** — Swap/Add exercise can no longer spawn two overlapping modals via a fast double-tap during the picker's fetch window; buttons disable while loading, unique-guard flag prevents the race, error handling added. Rest/timed-set/cardio-interval 5-second beeps switched from Web Audio tones (silently suspended by iOS mid-rest) to spoken numbers via the already-reliable Web Speech channel. 2026-07-04, pushed 84f9267.
- Edit logged sets in runner; PT notes always shown; client notes textarea
- Runner save lands on correct page per role (client → Workouts, PT → client profile)
- Client dashboard — hero "Up next" card (program + phase + week), goals, weight, upcoming events, PBs, recent sessions; PT branding header when set
- PT dashboard — hero card, stats strip, two-column grid, activity feed (coach-scoped, no solo bleed), compliance cards, goals due
- Solo dashboard — hero card, stats strip (hidden on mobile), two-column grid
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

- **Runner set-accuracy build (session 11, 2026-07-03) — built + Playwright-verified, NOT YET PUSHED.** Per-set target display fixed (table mode was hardcoded to always show set 1's prescription — now dynamic, tracking the current working set like the wizard already did); delete-a-set added (wizard edit sheet + table rows, neither existed before); live rep tally (current exercise's running reps vs. same exercise's total last session, updates per set); swap/add exercise now opens the **literal same modal** used in the workout builder (library picker + 1RM group + full set-target builder + notes + superset) via a parameterised `showAddExerciseToTemplateModal(templateId, runnerCtx)` — corrected from an initial simplified picker after Jake's live-test feedback that "the same modal" meant literal reuse, not a stylistically-matching rebuild. Tick checkbox redesigned (white → green on complete, was near-invisible); delete button redesigned (red "Delete" label, was a bare ✕). All four came from Jake live-testing the approved build in the same session — see LOG for the full before/after per item.
- **Programs picker "Filter workouts below" search — diagnosed, not fixed.** Confirmed via two independent live-DOM tests that the JS filtering mechanism works correctly; the real problem is a plain `<select>` gives no visible feedback until manually opened, so it looks broken. Connects to Jake's separate complaint that the assign-workout list will keep growing unmanageably. Recommended fix (awaiting Jake): replace with a tap-row list that live-filters — the same pattern this session just removed from the runner's picker in favour of literal modal reuse.
- **`deleteProgram()` orphan-cleanup — ✅ Done (66bf1fd, 2026-07-03).** See "What's working" above. Historical backlog note: the fix stops *future* debris, but the existing ~993-template backlog on the main coach account (found while researching this fix) is separate and still needs its own cleanup pass — same class as the 65 orphans cleaned 2026-07-02. Jake's calls so far: skip building a one-click replace/update-assignment flow for now (just prevent silent double-assign); when calendar integration is built, it should write real per-session `calendar_events` rows, not a lighter marker.
- **Runner redesign → Hevy-style table logger** — ✅ v1 SHIPPED 2026-07-02 (6e6402a). See "What's working" above. **Phase 2 not started:** extend the table pattern (or an equivalent) to cardio/timed/unilateral/%1RM exercises, which still use the old one-set wizard. Two known gaps in v1, not yet resolved: (1) superset auto-switch (wizard jumps to the paired exercise on log; table never auto-advances by design, so a superset pair just sits there until the PT/client manually taps Next — only matters if real templates use `supersetGroup`, not yet checked against Jake's actual data); (2) bodyweight-in-table is code-reviewed but not live-verified (didn't hit a live bodyweight exercise in this session's test data).
- **Runner bug fixes** — ✅ Done (61d8bc7, 2026-07-02): set-counter cap, rest-timer-before-render (×3 branches), beep window 3s→5s, audio-unlock on both entry points. Playwright 48/48 + CI green + deployed. Audio-unlock real-device check deferred by Jake — mobile-web audio is a known limit a native app (Capacitor) will resolve properly.
- **Runner audit (vs the "Hevy" consumer-app bar)** — findings banked to roadmap.md as build items: (1) strength inputs aren't pre-filled → every set is a full retype vs Hevy's 1-tap repeat (biggest gap); (2) no plate calculator; (3) background rest-timer alerts need PWA/native; (4) last-session strip is strength-only. Rationale lives in the LLM wiki `coachapp-product-strategy` + `coachapp-client-app-benchmarks` pages.
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
Each of the 8 module files has its own independent `?v=N`. Any commit that changes a module file must bump that file's own version in index.html — bump only the files that changed, not all 8. Current (live): core/dashboard/clients v=1 · programs v=8 · calendar-goals v=2 · workouts v=9 · runner v=9 · progress v=3.

### Strength table (Hevy-style runner v1)
`_isPlainStrengthExercise(ex)` gates the table vs. wizard: excludes cardio, any set with `timed`/`unilateral`/`intensityMin`, and carry/sled/lunge-name exercises (regex-matched, same list used elsewhere). `ex.tableRows` = `[{weight, reps, done}]`, lazily built by `_ensureTableRows` (seeded from `_prevSetsByIndex`, keyed by `set_number - 1` against `_runner.lastSession[ex.name]`). `toggleTableSet(i)` flips `done` and calls `_syncLoggedSetsFromTable`, which **rebuilds** `ex.loggedSets` from scratch each time (`filter(done).map({weight,reps})`) rather than pushing — verified safe: every consumer (`saveRunnerSession`, `showRunnerFinish`, header dots) only reads `.length`/iterates, never holds a stale reference. `renderRunnerLastSession` was extended (not just the table added) to retroactively backfill blank rows if the async last-session fetch resolves after the table's first paint — guarded against re-triggering itself (row already non-blank → skip → no re-render loop).

### Periodization — week_number / tier
`program_phases.periodization_type` (`'linear'`/`'undulating'`/null) + `periodization_config` (jsonb). `program_phase_workouts.week_number` (default 1) lets one phase hold distinct day/template assignments per week — phases that never use periodization just have week_number=1 rows, which the client calendar/workouts-page render as repeating every week (legacy behaviour, unchanged). `program_phase_workouts.tier` (`'heavy'`/`'moderate'`/`'light'`) is undulating-only, set per day-slot. `generatePhasePeriodization(phaseId, programId)` clones Week 1's templates into weeks 2..duration_weeks, recalculating only sets where `intensityMin`/`intensityMax` is set — everything else (reps, rest, tempo) is copied unchanged. Regeneration is idempotent (`_cleanupPhaseWeeksBeyond` deletes stale weeks + their templates, both master and any already-propagated client copies, before regenerating) — the same helper runs when a phase's `duration_weeks` is edited down. Client propagation: assigning a program clones week_number through (`_cloneTemplateForClient`/`_cloneProgramForClient`); if a client is *already* assigned when new weeks are generated, `generatePhasePeriodization` propagates to them too.

**%1RM vs RPE — not interchangeable (clarified 2026-07-03):** the phase-level Start/End % (or undulating tiers) only scales sets that were built with the **%1RM field** in the set editor (`intensityMin`/`intensityMax` on that set). Sets built with RPE/RIR instead are left completely untouched by design — periodization has nothing to scale on an RPE-based set, so the session-detail drawer correctly shows nothing extra for them. This is a per-exercise, per-set authoring choice in the set editor, not a phase-wide toggle. Found via Jake's test program: Phase 1 set to Linear 70→80%, but Bench/Squat/Row were all built with RPE, so nothing visibly changed — expected behaviour, not a bug.

### 1RM assignment-time check
`_getProgramOneRMStatus(programId, clientId)` scans Week 1 only (sufficient — generated weeks reuse the same exercise names) for %1RM-tagged sets, diffs against `client_1rms` (trim+lowercase exact match — same limitation as the rest of the app; exercise_id-based matching is a deferred future decision, see roadmap). `_renderOneRMQuickEntry(idPrefix, name)` is the shared toggleable kg/Epley-estimate row component — parameterized by idPrefix so multiple rows coexist. `window._missingOneRMExercises` holds the current checklist's missing-name list (index-matched to `mor-N` DOM ids) between refresh and save. `_refreshMissingOneRMs` is token-guarded (`_oneRMRefreshToken`) so a stale in-flight refresh can't overwrite a newer one if the PT changes the client/program selection quickly. Wired into both assign entry points (`showAssignProgramModal`/`showAssignProgramToClientModal`) — 1RM entries must be saved *before* the modal is removed, not after (DOM reads fail silently on a detached modal).

---

## Open to-dos for Jake

| Action | Priority |
|---|---|
| **Decide + build runner session autosave** — the real fix behind "lost all my session data" (2026-07-04 live incident): `_runner` lives only in memory until the final "Save workout" tap, so any crash/freeze/forced-reload mid-session still loses everything logged so far, even though the exercise-picker freeze that triggered this specific incident is now fixed. Needs a scoping conversation (local draft vs. periodic DB write vs. other). May fold together with the "workout-tracking visuals + data storage" item below. | **High** |
| **Improve workout-tracking visuals + underlying data model** — Jake wants a better look at how a workout is tracked in the runner, and to reconsider where/how that data is stored (added 2026-07-04). Not scoped yet — needs a sounding-board session; likely overlaps with the autosave decision above. | Medium |
| **Other git worktrees created before 2026-07-02 may still auto-start the dead "PT Dashboard" (PTHub) config instead of CoachApp** — today's fix removed it from this worktree + the parent seed template, but existing worktrees weren't swept (unlike the 18-worktree sweep done earlier today for the stale-path bug). Only bites if CoachApp's own server isn't already running when a worktree's preview auto-starts. | Medium |
| Check whether any of Jake's real workout templates use `supersetGroup` — the new strength table doesn't auto-advance to a superset partner the way the old wizard did (by design — table is "tap Next when ready"), so paired exercises need a manual tap between them now. Only worth fixing if real templates actually use supersets. | Medium |
| Live-verify a bodyweight exercise in the new strength table (code-reviewed, not tested against real bodyweight data this session) | Low |
| Run /deploy-check before next beta invite | **High** |
| **1RM exercise-name matching — exercise_id vs fuzzy string match** — big design decision, deferred to its own session (see roadmap.md). Today's build keeps name-matching in one shared helper so this swap is contained later. | Medium |
| **Orphaned/duplicate workout_templates backlog (~993 templates on the main coach account)** — `deleteProgram()` was fixed 2026-07-03 (66bf1fd) so this stops growing going forward, but the existing historical backlog still needs its own cleanup pass, same class as the 65 already cleaned 2026-07-02. A read-only diagnostic SQL was handed to Jake in an earlier session (per-template reference-count breakdown, scoped to his coach_id) — awaiting him to run it and paste results back before any DELETE is drafted. | Medium |
| **Programs picker "Filter workouts below" search needs a design decision** — mechanism confirmed correct (live-tested twice); the real gap is a plain `<select>` gives no visible feedback until manually opened, so it reads as broken, and the list will keep growing unmanageably as-is. Recommended: replace with a tap-row list that live-filters. Awaiting Jake's call. | Medium |
| **`client_1rms` may lack an INSERT policy for the `client` role** — a real insert attempt (client's own client_id) failed with an RLS violation during manual testing 2026-07-03. Pre-existing gap, not caused by this session's diff (`saveRunnerOneRM` already writes to this table in production) — needs a `pg_policies` check, not yet investigated. | Medium |
| Verify iOS Safari slide-in fix on real device — `inset:0` replaced with explicit `top/right/bottom/left` in v180, pushed. Test on iPhone and close if working. | **High** |
| Run Rowing/Running/SkiErg DELETE SQL in Supabase (safety check first — see below) | High |
| Update invite-client Edge Function to include PT logo in invite email HTML | Medium |
| Test My Progress Strength tab on live with real data | Medium |
| ICO breach notification procedure — document before beta | Medium |
| Verify `workout_template_exercises` RLS policy exists (check pg_policies for that table) | Medium |
| **Verify `program_phase_workouts` INSERT/UPDATE/DELETE RLS policies join through `program_phases.program_id → programs.coach_id = auth.uid()`** — this table has no `coach_id` column of its own, so ownership is transitive; the new "Create new workout auto-assigns to day slot" code (and the pre-existing `_quickAssignPhaseWorkout`) both insert a bare `phase_id` sourced from a DOM `data-` attribute and trust RLS entirely. Same ownership-chain shape as the `workout_template_exercises` check above — do both in one pg_policies pass. Surfaced by 2026-07-03 security review; not a regression, pre-existing pattern. | Medium |
| **Decide `exercises` table scoping consistency** — 7 of 8 query sites (incl. the new runner exercise-picker) read `exercises` with no `coach_id` filter; only `app-progress.js:98` scopes it. Either it's an intentionally shared/global library (then app-progress.js is the odd one out) or it should be coach-scoped everywhere. Data is non-PII (exercise names + muscle groups), so low urgency — but pick one and make it consistent. Surfaced by 2026-07-03 security review. | Low |
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
| app-programs v=9 / workouts v=10 | Duplicate week (unified `renderPhaseWeekGrid`, every week equally editable); fork-on-edit for shared workout templates (`_resolveEditableTemplateId`); `deleteProgram()` solo self-assignment fix + PT toast names blocking clients. 3-agent review caught + fixed a stale confirm-dialog wording gap. 3 new Playwright tests (59/59). **Pushed 2026-07-04 (730738a).** |
| app-programs v=6 / workouts v=8 / runner v=7 | Fixes from Jake's live program-build + runner test (13-item list): strength-table target-info bar restored + inline 0:00 rest timer + visible tick button; mid-workout swap/add exercise (session-only); voice-cue fix (real gesture-tied speak) + female voice; create-new-workout auto-assigns to its day slot; Edit button on the session-detail drawer; phase card shows periodization range; tempo field 4-char cap. Playwright 51 passed + 3-agent review + 2 new smoke tests. Also cleaned 65 orphaned duplicate templates from Jake's account via a safety-checked SQL delete (FK-enumeration → reference-check → self-guarded transaction). **Pushed 2026-07-03 (5438aac).** |
| app-runner v=6 | Hevy-style strength table (v1: plain strength only) — all-sets-visible SET/PREVIOUS/KG/REPS/✓ table replaces the one-set wizard for non-cardio/timed/unilateral/%1RM exercises; previous-session pre-fill; non-blocking rest timer; 2 new Playwright smoke tests. Verified via Playwright (48/48 full suite + 12/12 runner.spec.js) + 3-agent review before push. **Pushed 2026-07-02 (6e6402a).** |
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
