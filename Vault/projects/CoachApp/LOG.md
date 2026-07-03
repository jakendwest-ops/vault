# CoachApp — Session Log

Newest first.

---

## 2026-07-03 (session 12) — Runner build pushed + 2 scoping fixes + deleteProgram rewrite + GitHub Pages dual-workflow fix (programs v6→v8, workouts v8→v9, runner v7→v9) — PUSHED (1914e7b, dee1479, 66bf1fd)

**Context:** Continuing straight from session 11's uncommitted runner build. Ran a fresh multi-agent review this session (none had run yet against this specific diff), pushed it, then a routine code-review scan turned up two real scoping bugs, then built and shipped the long-standing `deleteProgram()` orphan-cleanup fix — this time backed by actually querying Supabase's FK cascade rules first instead of assuming. A GitHub Pages deploy failure Jake forwarded led to finding and fixing a root-cause infrastructure issue unrelated to the code itself.

**Done — runner build push (1914e7b):**
- 3-agent review (security/scoping, solo-mode correctness, duplicates+render-safety) on session 11's set-accuracy build — all three came back clean. Two agents independently flagged the same dead-code nit (`curSig`, an unused variable in `renderStrengthTable`) — removed it, reran Playwright (56/56) before pushing.
- One coverage gap noted, not fixed: `tests/runner.spec.js` has no solo-mode tests, though the role-sensitive code path (`showAddExerciseToTemplateModal`'s `isRunner` branch) itself checked out clean via manual trace.

**Done — 2 scoping bugs found + fixed (dee1479, runner v9 / programs v7):**
- `showLogSessionModal` (app-runner.js) queried `workout_templates` with **zero scoping** — no `coach_id`, no `client_id` filter at all. Its "Load from template" dropdown was pulling every coach's templates, not just the current one. Scoped it the same way `saveRunnerSession`/`saveWorkoutSession` already derive `coachId` (client's `coach_id`, falling back to `currentUser.id`).
- `deleteProgram()` (app-programs.js) deleted by `id` only, no `coach_id` ownership check — inconsistent with `deleteTemplate`'s equivalent guard. RLS likely blocked cross-tenant deletes regardless, but added explicit scoping for defense-in-depth.
- Both found during routine code review, not part of an existing feature. Verified live in preview (dropdown loads correctly scoped, no console errors) + Playwright 56/56.

**Done — `deleteProgram()` full rewrite (66bf1fd, programs v8):**
- **Researched actual Supabase FK cascade rules before writing any delete logic** (SQL Editor, `information_schema.referential_constraints`) rather than assuming, per sql-safety habit. Found: `programs→program_phases→program_phase_workouts` cascade automatically; `workout_templates→workout_template_exercises` cascades automatically; `client_programs→client_program_workouts` also cascades (queried live — **0 orphaned rows found**, confirming "Remove program from client" has been cleaning up correctly all along, DB-enforced not just app-code-enforced). The only real gap: `workout_templates` itself survives its `program_phase_workouts` row being removed (`SET NULL`, not `CASCADE`) — that's the actual leak.
- This simplified the build significantly versus the originally-proposed plan (no manual FK-order deletes needed for phases/phase-workout-slots/template-exercises — cascade handles all of it).
- New behavior: check `client_programs` for the program first — if any clients are assigned, show a toast and stop (no confirm dialog, nothing touched). Otherwise walk `program_phases → program_phase_workouts.template_id` to find the templates this program owns, delete those explicitly, then delete the `programs` row.
- Verified both paths against **real, not simulated, conditions**: block path called directly against Jake's actual "Test 1" program (2 real clients assigned) — confirmed it returned early with the correct toast message and the program's phase content was unchanged afterward. Cleanup path tested against a disposable fixture program+phase+template+phase-workout-slot created via the app's own Supabase client and torn down in the same session — confirmed all four rows gone after one call. Playwright 56/56.

**Done — GitHub Pages dual-workflow fix (no CoachApp code):**
- Jake forwarded a "pages build and deployment: Some jobs were not successful" failure email. Investigated and found the repo's Pages config was in **legacy "deploy from branch" mode** (`build_type: legacy`) — which runs GitHub's own native build-and-deploy workflow on every push, *completely independently of and redundant with* the repo's custom "Check & Deploy" Action (which also builds and deploys via `actions/deploy-pages@v5`). Both fire on every push; both can independently hit GitHub's transient "Deployment failed, try again later" infra error — which is exactly why the 66bf1fd push generated two separate failure emails for the same underlying hiccup, and why the "Check & Deploy" job also failed on this push in isolation (confirmed via `gh run rerun` — same commit, second attempt succeeded clean).
- **Got Jake's explicit approval before changing repo configuration**, then switched Pages source to Actions-only (`build_type: workflow`) via `gh api -X PUT repos/.../pages -f build_type=workflow`. Verified the live site was unaffected (still serving the correct version) immediately after.

**Bugs found + fixed:**
- `showLogSessionModal` unscoped query, `deleteProgram()` missing `coach_id` check — see above, the latter superseded by the same-session full rewrite.

**Decided:**
- GitHub Pages deploy source is now Actions-only — one deploy workflow going forward, not two.

**UNVERIFIED (banked):**
- The GitHub Pages source-switch fix is verified for this session (site unaffected, `build_type: workflow` confirmed via the API) but its durability across future pushes — whether the native workflow genuinely never fires again — is only provable by watching the next few pushes.
- The ~993-template backlog on the main coach account (surfaced while researching `deleteProgram`'s FK behavior) is separate from and larger than what this session fixed — `deleteProgram()` now prevents *new* debris of this kind, but the historical backlog still needs its own cleanup pass.

**Why:**
- Checking the actual FK cascade rules in Supabase before writing delete logic (rather than the originally-proposed manual multi-step cleanup) both simplified the build and resolved a previously-open uncertainty about whether "Remove program from client" was reliably cleaning up client data — confirmed yes, and DB-enforced, not just app-code-enforced, which is a stronger guarantee than assumed.
- The GitHub Pages fix follows the standing pattern of fixing root causes rather than re-explaining the same symptom every session: two workflows silently deploying redundantly would have kept generating confusing duplicate failure emails indefinitely otherwise.

---

## 2026-07-03 (session 11) — Runner set-accuracy build + swap/add modal unification + orphan-template diagnosis — NOT YET PUSHED (app-runner v7→v8, app-workouts v8→v9)

**Context:** Continuing session 10's same-day work. Jake gave a detailed PT + client-runner spec (program building, per-set target display, runner affordances) and approved scoping the runner "set-accuracy" bucket first, deferring program-assignment/calendar work. He then live-tested the build himself and returned two rounds of concrete, screenshot-driven feedback that reshaped the implementation.

**Done — runner set-accuracy (round 1, approved before building):**
- **Per-set target display fixed** — the strength table's target bar was hardcoded to always read `sets_json[0]`, so sets 2/3 with different %/RPE/rest/reps never showed their own prescription. Now dynamic: tracks the current working set the same way the wizard already did.
- **Delete a set** — wizard's logged-set edit sheet gets a Delete button; table rows get a delete affordance. Neither existed before.
- **Live rep tally** — running total of reps logged for the current exercise vs. the same exercise's total last session, updating on every set registered (table mode + wizard reps/unilateral sets).
- **Shared 1RM-aware exercise picker (v1)** — swap/add rebuilt to mirror the workout builder's "pick from library" pattern with a 1RM-lifts group, so a picked 1RM lift is linked for weight calculation. Session-only, unchanged from Jake's earlier decision.
- New skill: **feature-audit** — runs after every feature build: affordance/permission audit (should this be editable/deletable, is the right control present, does RLS actually allow it), a PT lens and a gym-user lens, ending in proof (Playwright + preview) not claims. Registered in hello-claude + memory.
- Playwright: 55/55 (5 new tests). Manually verified live in the preview (dynamic target bar, captions, tally, row delete, wizard delete, 1RM optgroup). Methodology note banked: `_runner` is a page-scoped `let` declared in app-workouts.js, not a `window` property — `window._runner = ...` silently no-ops; must assign the bare identifier.

**Done — round 2, from Jake's live-test screenshots (four corrections):**
- **Per-set correlation redesigned** — dropped the per-row caption text ("ugly UI" per Jake); the row for the set currently being worked is now highlighted instead, so the target bar's numbers are visually tied to a specific row rather than described in text.
- **Tick checkbox redesigned** — was effectively invisible (faint border against the row background); now a plain white box with a visible 2px border, turns `var(--success)` green with a white check on completion (reused the existing green token, not a new colour).
- **Delete button redesigned** — was a bare "✕" with no indication of what it did; now a small red-tinted box reading "Delete" (`var(--danger-light)`/`var(--danger)` — existing tokens).
- **Swap/Add now open the literal same modal as the workout builder** — round 1 had built a simplified select-only picker matching the builder's *style*; Jake corrected this to mean literal component reuse. `showAddExerciseToTemplateModal` in app-workouts.js is now parameterised (`runnerCtx = {mode}`) so the runner's Swap/Add buttons open the exact same modal (library picker + 1RM group + full set-target builder + notes + superset), with a runner-specific confirm handler (`_confirmRunnerExerciseFromModal`) that applies the result to `_runner.exercises` in-memory instead of writing to `workout_template_exercises` — keeps swap/add session-only. Verified byte-identical modal markup between the two entry points in a Playwright test.
- Playwright: 56/56 full suite (tests rewritten to match the real modal's fields, not the old simplified picker).

**Diagnosed, not yet built (awaiting Jake):**
- **Programs picker "Filter workouts below" search** — mechanism confirmed correct via two independent live-DOM tests (isolated, and on Jake's real "[E2E] Inline Grid Test" program); the actual problem is that a plain `<select>` gives no visible feedback until manually opened, so typing "looks like it does nothing." Ties directly to Jake's separate complaint that the list will keep growing unmanageably. Recommended: replace the select+search pattern with a tap-row list that live-filters — the same pattern just replaced in the runner's picker, ironically; flagged the tension honestly rather than silently.
- **`deleteProgram()` orphan root cause reconfirmed** — enumerated every FK reference to `workout_templates` (`workout_template_exercises.template_id`, `program_phase_workouts.template_id`, `client_program_workouts.workout_template_id`); confirmed `deleteProgram()` (app-programs.js:755) deletes only the `programs` row, never its cloned master/periodization-week templates. Proposed: clean up owned templates on delete, and block deletion (clear message) if clients are currently assigned rather than silently failing or orphaning client data. **Awaiting Jake's decision on scope.**
- **Read-only diagnostic SQL handed to Jake** (not yet run) to see exactly which `workout_templates` are orphaned vs. in-use, before any DELETE is drafted — sql-safety protocol, no destructive SQL written speculatively.
- **New RLS gap found in passing:** a real `client_1rms` insert attempt (client role, own client_id) failed with an RLS violation during manual testing. Pre-existing — `saveRunnerOneRM` already writes to this table in production, untouched by this session's diff — but worth a dedicated `pg_policies` check; not yet investigated further.

**Decided:**
- "The same modal" in Jake's spec means literal component reuse, not a stylistically-matching simplified rebuild — corrected mid-session after his explicit screenshot feedback.
- Runner swap/add stays session-only (reconfirmed, unaffected by the modal-unification change).
- Program-assignment replace/update flow is deferred; calendar integration (when built) writes real per-session events, not a lighter marker.

**UNVERIFIED (banked):**
- All of this session's runner changes are Playwright + live-preview verified but **not pushed** — no multi-agent code review has run yet, and Jake has not said commit/push. Sitting locally: `index.html`, `js/app-runner.js` (v8), `js/app-workouts.js` (v9), `tests/runner.spec.js`.
- Whether the fuller swap/add modal (Type/Notes/Superset/full set-target builder, not just a name field) feels like too much friction for a live mid-workout swap versus the simpler version originally built — banked as a prediction, only real gym use will tell.

**Why:**
- Jake's screenshot-driven correction round validated the new feature-audit skill's premise from the opposite direction: even with Playwright green and a live-preview pass done in the same session, his actual gym-context testing caught four real UX issues automated checks don't reason about (visual invisibility, ambiguous icon meaning, textual vs. visual correlation, literal-vs-equivalent component reuse). None were logic bugs — all four were judgment calls about what a real user sees and understands.

---

## 2026-07-03 (session 10) — 13 fixes from Jake's live test run + orphaned-template cleanup + wiki docs (programs v6, workouts v8, runner v7) — PUSHED (5438aac)

**Context:** Jake ran his own end-to-end test (create a program → generate 4 weeks → run it) and reported 13 numbered issues via screenshots. Traced all 13 to code, bucketed them (3 quick fixes / 4 confirmed bugs / 4 needing his decision / 2 new features), got his calls on the decision bucket, then built.

**Done — runner (v6→v7):**
- **Target-info bar restored to the strength table** — the v1 table redesign had dropped the rep range/RPE-RIR/tempo/rest/%1RM prescription. Extracted the wizard's target-column logic into shared `_buildTargetCols`/`_renderTargetBarHtml` and rendered it above the table (incl. the %1RM→kg target + "set your 1RM" banner).
- **Rest timer moved inline + reformatted** — for table-mode exercises the rest timer now renders as a plain `0:00` countdown bar *inside* the target section (kept `id=rest-timer-overlay` so existing tests still target it), instead of a `position:fixed;top:0` overlay that was covering the runner's own header. Wizard mode keeps the old floating overlay. `startRestTimer` now skips the floating `renderRestTimer()` in table mode and does a bounded `renderRunner()` on completion.
- **Voice cue fixed** — `_unlockSpeech()` only called `speechSynthesis.cancel()`, which never registers the user gesture, so the 10s "10 seconds" cue was silently blocked. Now does a real (silent) gesture-tied `speak()`. Added `_pickFemaleVoice()` so `speakCue` uses a female voice.
- **Visible tick button** — the unchecked set-complete ✓ button had no border and matched its row background (invisible); added a `1.5px` outline.
- **Swap exercise mid-workout** (session-only) — `showExercisePicker('swap')` → `_swapRunnerExercise`: replaces the current exercise in-memory, clears its logged/table state, refetches previous-session data under the new name.
- **Add exercise mid-workout** (session-only) — `_addRunnerExercise`: appends a new plain-strength exercise and jumps to it. Neither swap nor add writes to `workout_templates`.

**Done — programs (v5→v6) + workouts (v7→v8):**
- **Create-new-workout auto-assigns to its day slot** — `saveNewTemplate()` now inserts the `program_phase_workouts` row (with race-check) back into the `ctx.phaseId`/`ctx.dayOfWeek` it was created from, instead of leaving the slot empty and forcing the coach to re-find and re-assign.
- **Edit button on the session-detail drawer** — `openSessionDetail(templateId, name, ctx)` now takes a ctx and shows an Edit button (gated `role !== 'client'`) routed through `openTemplate` so the propagate-to-all prompt still fires. All 3 call sites pass appropriate ctx.
- **Phase card shows periodization range** — "Linear (70→80%)" / undulating tier %s from `periodization_config`, not just the type name.
- **Clearer day-column search placeholder** ("Filter workouts below…").
- **Tempo field capped at 4 chars** (workouts v8).

**Done — data cleanup (Jake's account, via safety-checked SQL he ran in Supabase):**
- Diagnosed ~114 duplicate standalone templates in the picker. Ran the full sql-safety path: FK-enumeration first (caught that `workout_template_exercises` also FKs to templates and must be deleted as a child) → per-name reference breakdown (orphaned vs used-in-program/client-plan/log) → a single self-guarded transaction that deletes only orphans (NOT EXISTS against all 3 "in use" tables) plus their exercise children. **65 orphaned templates deleted**, all in-use copies preserved.

**Done — LLM wiki (separate from this Vault):**
- Authored two architecture/reference pages at Jake's request: `coachapp-programs-architecture` (two-level Program/Client-Plan model, data model + the program_id/client_id/generated_from_phase_id triplet, build grid, periodization incl. %1RM-vs-RPE, assignment/cloning, propagation, known gaps) and `coachapp-runner-architecture` (the `_runner` object, table vs wizard modes, rest timer + voice cue, pre-fill, swap/add, save path, known gaps). Updated `coachapp-workflows` (pre-fill + swap/add now marked shipped), index, log.

**Bugs found + fixed (during verification, not shipped broken):**
- **Exercise-picker modal rendered behind the runner** — `.modal-overlay` is z-index 200, the runner's fullscreen div is 300, so the picker rendered underneath and Playwright's clicks were intercepted by the runner. Caught by the new smoke tests (not review). Fixed with an explicit `z-index:1000` on the picker overlay.
- **Voice cue** (above) — a real bug the code review's premise surfaced, then confirmed empirically (6 voices present in the browser, so not an environment gap — the priming was the fault).
- Review agent flagged the picker's `exercises` fetch had no error handling — added a toast on failure.

**Tests:** Playwright pre-push hook full suite **51 passed** (1 flaky — a Supabase-timing phase-render assertion, passed on retry, unrelated to the diff). 2 new runner smoke tests (swap + add exercise pickers). 3-agent review (security/scoping, solo-mode, duplicates/render-safety) — all clean or pre-existing; the render-loop trace on the new inline rest timer came back bounded.

**Decided:**
- Swap and add exercise are **session-only** (Jake's call) — they change today's log, never the saved template.
- %1RM vs RPE is not interchangeable: periodization only scales %1RM-tagged sets; RPE sets are untouched by design (documented, not a bug — it confused a test).
- The `deleteProgram()` orphan bug is **confirmed live** — the deleted `— W2/W3/W4` clones were its debris. Bumped to High; fix before building more programs.

**UNVERIFIED (banked):**
- The runner UI changes (target bar, inline rest timer, swap/add) are verified via Playwright + 3-agent review, not a real-device gym session — banked as a prediction, mirrors the v1 table's pth-070.
- CI "Check & Deploy" job failed on a transient GitHub Pages deploy race; the site is confirmed live on v7 via curl, and the native pages-build-deployment workflow succeeded. Not a code failure — watch whether it recurs on the next push.

**Why:**
- Bucketing the 13 items before building (and getting Jake's calls on the 4 decision items) avoided building the wrong thing for the ambiguous ones — e.g. #6 turned out to be no-bug-at-all (his test sets used RPE, so periodization correctly did nothing).
- The SQL cleanup followed sql-safety to the letter precisely because it was 100+ irreversible deletes on Jake's real account — FK enumeration first is what caught the `workout_template_exercises` child dependency that a name-based delete would have failed on.

---

## 2026-07-02 (session 9) — Runner redesign v1 shipped: Hevy-style strength table (runner v5→v6) — PUSHED (6e6402a)

**Done:**
- **Preview tooling fix (no CoachApp code):** diagnosed why the preview flipped to serving PTHub — this worktree's `.claude/launch.json` listed a dead "PT Dashboard" (PTHub) config *before* the CoachApp config, and it auto-started once the other chat's real CoachApp server (port 3001) had shut down. Removed the dead PTHub config from this worktree's launch.json **and** the parent seed template (`C:/Users/jaken/Claude/.claude/launch.json`), so no future worktree can inherit it. Did not sweep other pre-existing worktrees — banked as a to-do.
- **Runner redesign v1 — Hevy-style strength table.** Plain-strength exercises (excludes cardio, timed, unilateral, %1RM-tagged sets, carry/sled/lunge-name exercises) now render an all-sets-visible table (`SET · PREVIOUS · KG · REPS · ✓`) instead of the one-set-at-a-time wizard. Previous-session values pre-fill KG/REPS for 1-tap repeat sets (reuses `fetchRunnerLastSession`, keyed by `set_number`). Tap ✓ completes a set — fires the rest timer as a non-blocking bar while the table stays visible/editable underneath (the core design difference from the wizard's old blocking "Resting…" placeholder). Uncheck to undo. "+ Add set" appends a row. Free-edit any row, any time, no forced order — no auto-advance (PT/client taps "Next exercise" manually). New functions in `js/app-runner.js`: `_isPlainStrengthExercise`, `_prevSetsByIndex`, `_ensureTableRows`, `_syncLoggedSetsFromTable`, `toggleTableSet`, `addTableRow`, `renderStrengthTable`; `renderRunnerLastSession` extended to retroactively backfill blank rows if last-session data arrives after the table's first paint. Reuses the exact `{weight, reps}` shape `saveRunnerSession` already expects — zero save/schema changes. `app-runner.js` v5→v6.
- 2 new Playwright smoke tests in `tests/runner.spec.js` (table renders with correct columns; checking a set logs it + starts rest without leaving the table) — both ran real assertions against the E2E account's actual assigned template, not skip-paths.

**Bugs found + fixed:**
- Strength-table checkbox tap target was 32×32px on first build — below the 44×44 standard minimum, and this is the single most-tapped element in the whole redesign (once per set, real gym conditions, sweaty/tired hands). Caught via live testing in the preview before declaring the feature done; bumped to 44×44 and reverified via exact pixel measurement.

**Decided:**
- v1 scope holds exactly as approved: plain strength only. Cardio/timed/unilateral/%1RM stay on the existing wizard as phase 2, not yet scoped.
- Superset auto-switch (wizard behaviour: completing a set on a superset exercise auto-jumps to its pair) is **not** replicated in the table — the table's "no auto-advance, tap Next when ready" design makes this the correct call, but it means a superset pair now needs a manual tap between them. Not yet checked against whether Jake's real templates use `supersetGroup`.

**UNVERIFIED (banked):**
- Bodyweight exercise in the new table — code-reviewed (the `ex.bodyweight` branch exists and mirrors the wizard's BW display), but no live bodyweight exercise existed in this session's test data to exercise it against.
- Whether any of Jake's real templates use `supersetGroup` (relevant to the superset gap above).
- Whether other pre-existing worktrees still carry the dead PT-Dashboard launch config (only this worktree + the seed template were fixed this session).

**Testing/review this session:**
- Full Playwright suite: 48/48 green (zero regressions across PT/client/solo roles).
- `tests/runner.spec.js` alone: 12/12 green (10 pre-existing + 2 new).
- 3-agent code review (security+scoping, solo-mode correctness, duplicates+PII), each independently verifying rather than taking claims at face value — all three returned SHIP with no blocking findings. Notably, the duplicates+PII agent was specifically asked to trace whether the modified `renderRunnerLastSession` (which now conditionally calls `renderRunner()` again) could create an infinite render loop — traced the full call chain and confirmed it's bounded to exactly one extra render per exercise, not recursive.
- Pre-push hook: 21/21 smoke tests + all static checks (only the expected sudo-gating hardcoded-email WARN). CI green, GitHub Pages deployed.

**Why:**
- The tap-target bug is exactly the kind of thing code review can't catch (it's a design/UX defect, not a logic bug) — only caught because live testing in the preview was done before declaring the feature done, consistent with the standing "verify before reporting done" rule.
- Multi-agent review scope was deliberately narrow (3 fixed angles) rather than open-ended — each agent was asked to independently verify specific claims (e.g. "are there really zero new DB queries") rather than just skim and vibe-check, which is what surfaced the fully-traced render-loop proof rather than a hand-wave.

---

## 2026-07-02 (session 8, part 5) — Four recurring "workaround every session" problems fixed at the source — NO COACHAPP CODE

**Context:** Jake's throughline all session — "this is the same 'keep working around it every session' problem as the launch.json path." Each fix converted a per-session patch into a root-cause fix + a self-check. No CoachApp module files touched (cache-bust + Playwright N/A this session).

**Done — 4 permanent fixes:**
1. **Preview stale-path (4th occurrence) → fixed at the seed.** Root cause: new worktrees are seeded by copying `.claude/` from the parent template `C:/Users/jaken/Claude/.claude/launch.json`, which itself held the stale `C:/Users/jaken/coachapp` root. Corrected the parent template + all 18 stale worktree copies. Future worktrees inherit the correct `OneDrive/coachapp` path. Memory `feedback_preview_verify.md` updated with the root cause.
2. **`/ingest` misfire → dedicated `/wiki-ingest` skill.** The framework family (brief/checkup/ingest/review/save/setup/export/ultraplan) are **slash commands** in `Claude/.claude/commands/`, active because sessions run with cwd = a worktree of the Claude framework repo. The LLM wiki ingest had no trigger of its own, so `/ingest` could only ever hit the framework pipeline. Created global skill `wiki-ingest` (thin — points at the wiki's own CLAUDE.md) + a self-correcting pointer in `ingest.md`.
3. **Token re-reading → `wiki/sources.md` manifest.** Every raw file's fate (ingested→where / skipped / discarded) in one machine-readable ledger; wired as read-first/update-last into the wiki CLAUDE.md workflow + the wiki-ingest skill. Reconstructed ground truth once from the citations already inside the wiki pages.
4. **`/save` + `hello-claude` golden path.** Framework `save.md` → `vault-save.md`, **committed** (`f6f3c48`) because `.claude/commands/` is git-tracked (a disk-only rename would resurrect in every worktree checkout); references updated in CLAUDE.md/conventions.md/setup.md; CoachApp `save` skill now chains `vault-save.md` as its final step so predictions/voice/ledger still fire. Recycled a stale full copy of all 8 commands in the `epic-murdock` worktree; removed an empty `coachapp/.claude/skills/` drift-magnet dir (the save skill's step 5 had pointed at it as "the skills location" — corrected to global). **Added a daily golden-path self-check to hello-claude Step 9** (one `find` sweep, self-repair without approval on regression).

**Done — LLM wiki batch-3 ingest (via the new `/wiki-ingest`):**
- Triaged 55 raw clips: discarded 8 Web Clipper re-clips/failed captures (Recycle Bin, verified by URL+size+diff), skipped 2 Google search-results pages, kept 13 already-fed primaries as provenance, **ingested 32 new sources**.
- New page **`coachapp-spreadsheet-competition.md`** (the ~20-thread Google-Sheets-vs-apps cluster: why PTs choose sheets, the RPE-auto-load power ceiling, where sheets break, the "program in a sheet / deliver an app" hybrid that is CoachApp's native shape). Appends to competitors (FitFocus + QuickCoach share a parent; FitFocus markets the exact client-experience wedge), competitor-pitfalls (won't-install-another-app; per-client pricing; coach-chosen demo videos), product-strategy (zero-install onboarding as underpriced advantage; "app for coaches who'd otherwise use sheets" positioning). Index/log/manifest updated.

**Decided / reinforced (strategy, from the sheets batch):**
- The real coach-side competitor is **Google Sheets**, not other apps. The switch trigger is ~5–15 clients. Positioning candidate: **"the app for coaches who'd otherwise use sheets."**
- A competitor (FitFocus) is already marketing the client-experience wedge ("the platform your clients will actually open") — the wedge is validated but the window isn't infinite, which reinforces the runner redesign as the right next build.

**Memory/skills touched:** `feedback_preview_verify.md`, `feedback_skill_golden_path.md`, `reference_llm_wiki.md`, `MEMORY.md` index all updated; new `wiki-ingest` skill created + registered in hello-claude standing behaviours + save step 5. Cleared the "skill-name collision" open to-do from STATUS.

**UNVERIFIED (banked):**
- The four self-checks are proven for *this* session (golden-path sweep returns exactly 2 artifacts; `/save` dispatched correctly to the CoachApp skill) but their *durability across a fresh future session/worktree* is by design unverifiable until the next one — captured as predictions instead.

**Why:**
- Every fix followed Jake's explicit template: don't patch the symptom in this worktree, find the one source that keeps re-emitting it (a seed template, a missing trigger, a git-tracked file), fix that, and add a cheap standing check so the fix can't silently rot.

---

## 2026-07-02 (session 8, part 4) — Runner redesign decided (Hevy table logger); Hevy screenshots ingested to wiki — NO CODE SHIPPED

**Done:**
- Session-start ritual: code review clean, CI green (28595278506), working tree clean, module versions confirmed (core/dashboard/clients v1 · programs v5 · calendar-goals v2 · workouts v7 · runner v5 · progress v3).
- Studied a 27-image Hevy iOS screenshot batch (Jake's, from a live app walkthrough). Key reference: the live logger is an **all-sets-visible table** (`SET · PREVIOUS · KG · REPS · ✓`) with previous values pre-filled → 1-tap repeat sets, inline rest timer (-15/+15/Skip), set-type sheet, Workout Settings toggles (Plate Calculator free, Warm-up Calculator PRO-gated).
- Ingested the batch into the LLM wiki (separate from this Vault): archived images to `raw/hevy-screenshots-2026-07-02/`, merged a "Hevy logger UI — teardown" section into `coachapp-client-app-benchmarks.md`, recorded the redesign decision in `coachapp-product-strategy.md`, logged it. Flagged that the images had been left on the Desktop, never actually in `raw/`.

**Decided (runner redesign — the next build):**
- Move the workout runner from the current **one-set-at-a-time wizard** to a **Hevy-style all-sets-visible table**: `SET · PREVIOUS · KG · REPS · ✓`, previous values pre-filled, tap-✓ to complete a set, rest timer fires on the ✓ tap.
- **v1 scope = plain strength (weight × reps) sets only.** Cardio / timed / unilateral / %1RM keep the current per-set flow and fold in as a phase 2. ("Build v1 and we'll see how it performs.")
- **Editing model:** free-edit any row any time (no forced order); rest timer triggers on the ✓ tick, not a sequential LOG.
- **PREVIOUS column** keyed by exercise name + set index, reusing existing `fetchRunnerLastSession` logic.
- **No paid gating** on any of it — inverse of Hevy's PRO wall; consistent with CoachApp's all-in positioning.
- Current runner lives in `js/app-runner.js` `renderRunner()` (~line 95); it already handles far more than Hevy (cardio pace/HR/distance, timed countdown, unilateral L/R, %1RM banner, RPE/tempo/rest chips) — that richness is exactly why v1 is scoped to strength only.

**Not done (deferred to next session, by Jake's call):**
- No consolidated build plan written yet, no code. Jake wants to start the build fresh in a new session.

**Bugs found + fixed (tooling):**
- `/ingest` and `/save` bare-name slash commands dispatch to a **global framework skill family** (brief/checkup/ingest/review/save/setup/export/ultraplan — the one that expects `System/conventions.md` + `Vault/ledgers/`), NOT the CoachApp/LLM-wiki flows. `/ingest` misfired to the framework skill mid-session; ran the wiki ingest by hand instead. Ran this save from its explicit path to avoid the same misfire. See to-dos.

**Why:**
- The redesign is a targeted runner rework, not an app rebuild — consistent with the product-strategy "redesign the flow, don't rebuild the app" caveat. Strength-first keeps v1 shippable without touching the hard cardio/timed/unilateral cases.

---

## 2026-07-02 (session 8, part 3) — Shipped the runner fixes; competitor research → product strategy; runner audit (workouts v6→7, runner v4→5) — PUSHED (61d8bc7)

**Done — CoachApp:**
- Verified and pushed the 4 runner fixes that had been sitting uncommitted since part 2: set-counter cap (kills phantom "Set 5 of 4"), rest-timer-started-before-render in all 3 log branches (kills the redundant double-display + phantom next-set input), beep window widened 3s→5s, audio-unlock on both runner entry points. Full discipline: Playwright 48/48 green → pre-push hook (all static checks + 19 smoke) green → CI green → GitHub Pages deployed. This clears the long-standing "runner fixes uncommitted" High to-do.
- Ran a runner audit against the consumer-app ("Hevy") bar. Findings: (1) **strength inputs aren't pre-filled** → the client retypes weight+reps every set vs Hevy's 1-tap repeat — biggest gap; (2) **no plate calculator**; (3) rest timer can't truly alert when the tab/phone is backgrounded (web limit → native app); (4) last-session strip is strength-only. Items 1 & 2 added to roadmap.md as build items.

**Done — LLM wiki (separate knowledge base at OneDrive\Documents\LLM wiki, not this Vault):**
- Web Clipper live-diagnosed (Edge v1.7.0, had never been configured), Jake configured it, verified end-to-end (a real clip landed in raw/). Registered the Vault as a second Obsidian vault so build tracking is browsable alongside the wiki.
- Ingested an 11-clip competitor/market-research batch into 4 cross-linked strategy pages + 2 new guide pages. Full detail is in the wiki's own log.md.

**Key strategic finding (banked to wiki + project_coachapp.md):**
- CoachApp's intended "all features, no tiers" differentiator is **already owned by PT Distinction** (4.7 Trustpilot, markets it hard). Sharper wedge from the research: a coach platform with a **Hevy-quality client app + honest/independent-billing trust** — client-experience quality, not feature count. Jake accepted this and is willing to not-rush beta to design from research rather than personal preference.

**Deferred (by Jake's call):**
- Audio-unlock real-device check — mobile-web audio is a known limitation that a **native app (Capacitor, roadmap "future")** would resolve properly. Not treated as a pressing bug.

**Cleared this session:**
- Runner fixes uncommitted → shipped (61d8bc7).
- E2E test-data gap (solo client record + assigned program on the E2E account) → resolved: all 9 solo-account tests and the program-accordion tests **passed (not skipped)** in the 48/48 run, which requires that data to exist. Removed both to-dos.
- Obsidian wiki 5-part follow-up → done (clipper checked+configured, usage scenarios documented, self-improvement guide created, new-project CLAUDE.md clarified, targeted novice quality pass done).

**Tests:** Playwright 48/48 full suite + 19/19 pre-push smoke. Zero console errors.

**Why:**
- Uncommitted work of any origin gets the full pre-push discipline (Playwright + review), not a diff-read guess — standing rule.
- The competitor research reframed the product differentiator: significant enough for a strategy layer + flow/priority rethink, but not a pre-beta rebuild (evidence is directional — biased competitor pages + Reddit anecdotes).

---

## 2026-07-02 (session 8, part 2) — LLM Wiki built + runner bug investigation (fixes applied, NOT pushed)

**Done — Obsidian "LLM Wiki" knowledge base:**
- Renamed `OneDrive\Documents\LLM wiki - coachapp` → `LLM wiki` and re-scoped it from CoachApp-only to a single wiki covering all of Jake's projects, CoachApp registered as the first one, following the "brief/checkup/ingest" skill family's apparent one-wiki-many-projects design
- Replaced the misplaced graphify-mirror CLAUDE.md with the real Karpathy LLM Wiki pattern content (ingest workflow, page format, citation rules, question-answering, lint, rules), Purpose section adapted for multi-project use
- Wrote 9 novice-oriented guide pages + 1 visual roadmap page (`guide-start-here`, `guide-glossary`, `guide-what-is-obsidian`, `guide-what-is-claude`, `guide-how-they-work-together`, `guide-daily-workflow` with a Mermaid decision flowchart, `guide-web-clipper`, `guide-new-project-setup`, `guide-share-with-a-friend`, `guide-coachapp-roadmap`), all cross-linked and registered in `wiki/index.md` and `wiki/log.md`
- Also produced an in-chat visual (show_widget) version of the CoachApp roadmap for the same session

**Bugs found + fixed (tooling/infra):**
- An empty Obsidian vault (`.obsidian` + `Welcome.md`) got accidentally created inside the actual `coachapp` git repository while Jake was switching vaults — untracked, not gitignored, real risk of polluting the public GitHub Pages repo. Deleted (Recycle Bin).
- Two duplicate/stale CLAUDE.md copies and dead Obsidian vault registrations (in `obsidian.json`, pointing at renamed/deleted folders) cleaned up. All traced to "CLAUDE.md" meaning two entirely different files depending on context (code-project graphify rules vs. wiki ingest rules) — now documented explicitly so it doesn't recur.

**Investigated — workout runner, from a live gym-test bug report with screenshots:**
1. **Rest timer redundant double-display + phantom "Set 5 of 4"** — root cause found: `renderRunner()` was being called *before* `startRestTimer()` in three separate branches of `logRunnerSet()` (strength hitTarget, cardio interval-complete, and the between-set case), so the page rendered a "next set" input for a set that didn't exist, with the static target-info row still showing underneath the live rest overlay. Set counter (`setNum = loggedSets.length + 1`) had no upper bound.
2. **Missing beeps** — code already had a 10s voice cue, 3/2/1 closing beeps, and a distinctly longer/higher finish beep; the audio-unlock call only fired on the first LOG tap, giving minimal head start before the first rest period needed it.
3. **Exercise navigation + per-session extra set** — turned out to **already be fully built** (`runnerJumpTo`, `runnerGoBack`, `skipToNextExercise`, `addExtraStrengthSet`/`addExtraCardioSet`, the latter confirmed session-only via an in-memory `targetSets++`, never touching the DB template) — likely just never reachable/visible because bug #1 was firing at the exact moment the "+ Add extra set" button was meant to appear.

**Done — 4 fixes applied to `js/app-runner.js` + `js/app-workouts.js` (cache-bust already bumped: workouts v6→7, runner v4→5):**
- Capped `setNum` at `targetSets` so the counter can never overshoot
- Reordered all 3 `logRunnerSet()`/cardio-interval branches so `startRestTimer()` (which sets `_restInterval`) runs before the corresponding `renderRunner()` — fixes both the phantom set and the redundant double rest display
- Widened the beep window from `restRemaining <= 3` to `<= 5`
- Added `_unlockAudio()`/`_unlockSpeech()` to both real runner entry points (`startWorkoutRunner` and `launchRunner` — confirmed two separate paths exist, one bypasses the other) so the AudioContext gets maximum time to resume before the first rest period

**NOT DONE — session ended before verification:**
- Playwright suite not run against these specific changes
- Multi-agent code review not run
- Not committed, not pushed
- Attempted a live manual test as the E2E client account and hit a data gap: not one standalone template belonging to that account's coach has any exercises attached — nothing usable to test the runner against. Banked as its own to-do (merged into the existing "assign a program to the Playwright test client" item).
- Audio-unlock fix specifically cannot be confirmed via automated testing — needs a real device check once live

**Why:**
- Jake explicitly said "please investigate" rather than "please fix" — root-caused before touching code, consistent with his standing preference for diagnosis before action.
- Live manual browser testing accidentally authenticated against Jake's real personal account first (a stale persisted session, not something this session created) — caught before any test data was written to it, switched to the dedicated E2E client account instead. No real data touched.

---

## 2026-07-02 (session 8, part 1) — Verified and pushed pre-existing uncommitted work (app-programs v4→5, workouts v5→6, runner v3→4, progress v2→3) — PUSHED (0a0f89f)

**Found:**
- Session started with 6 files already modified in the coachapp working tree (index.html + 5 js/test files) — not documented in any prior LOG entry, authorship/origin unknown. No `/save` had been run for whatever session produced them.

**Done (verified, not authored, this session):**
- Session history (client Workouts page + PT client-profile Workouts tab) now collapses behind a toggle by default instead of always showing the full list, reusing the existing `toggleClientPhase` helper
- `loadAllPhaseWorkouts` batched into one query across all phases instead of N sequential round-trips per phase
- `delete1RM` / `deleteWorkoutLog` fixed to re-render the correct DOM target after delete (previously could target a stale/wrong element depending on calling context)
- `renderWorkoutTemplates` — removed the "group templates by program" section
- `renderClientWorkouts` — workout_logs query capped at 20 rows

**Verification before push:**
- Playwright: 48/48 passed (suite grew 47→48 — one new test covers the collapse/expand behaviour), zero console errors
- 3-agent code review: security/scoping (clean — batched query, template-list query, and `.limit(20)` all preserve coach_id/client_id scoping), solo-mode correctness (clean — fully traced, collapsible history confirmed identical for client/solo, PT-only variant confirmed unreachable from solo), duplicates/regressions (clean — confirmed via `git log -S byProgram`/`programMap` that the removed "group by program" section was broken since its introduction on 2026-06-26 (commit `3a7a62a`) and never once populated its grouping map in its entire history, so nothing that ever worked was removed)
- Pre-push hook: all static checks + 19 smoke tests green (only the known/expected sudo-gating hardcoded-email WARN)

**Bugs found + fixed (tooling, this session):**
- Worktree's own `.claude/launch.json` had the same stale-path bug as 2026-07-01 (`C:/Users/jaken/coachapp` instead of the OneDrive path) — third occurrence of this exact class of bug this project has hit
- `playwright` skill's own SKILL.md had the same stale path hardcoded in its run instructions, plus a stale test count ("X/26" — suite is now 47-48). Fixed both.
- An empty Obsidian vault was accidentally created inside the coachapp git repo itself (`coachapp\CoachApp\`, untracked, not gitignored) while switching vaults in Obsidian — real risk of vault config or future notes getting committed and pushed to the public GitHub Pages repo. Deleted (Recycle Bin, not permanent).
- Two duplicate/stale CLAUDE.md copies and two dead Obsidian vault registrations (pointing at renamed/deleted folders) cleaned up — all traced back to confusion over "the CLAUDE.md file" meaning two entirely different things (graphify rules for the code project vs. a separate wiki-system template).

**Decided:**
- Set up a new Obsidian-based "LLM Wiki" (Andrej Karpathy pattern: `raw/` sources → Claude-maintained `wiki/` pages → `index.md`/`log.md`) at `C:\Users\jaken\OneDrive\Documents\LLM wiki`, separate from this project-tracking Vault. Vault stays for day-to-day session tracking (STATUS/LOG/roadmap, written automatically every session); the new wiki is for curated knowledge from source documents Jake feeds in on purpose. Designed as one wiki covering all of Jake's projects/interests, not one wiki per project — CoachApp is registered as its first project in `wiki/index.md`, with room for others (e.g. a Japan trip, which was the template's original example content) alongside it.
- Uncommitted work of unknown origin gets the same pre-push discipline as anything written in-session — Playwright + multi-agent review before push, not a read of the diff and a guess. This is what turned "looks like it might be a regression" (the template-list simplification) into a certain "confirmed dead-code removal, verified via git history."

**Why:**
- The stale-path bug recurring a third time (worktree launch.json twice now, plus the playwright skill itself) confirms this is a systemic pattern worth naming, not a one-off — any skill or config file with a hardcoded filesystem path is a candidate for silent drift whenever the project moves or a new worktree is created.
- The accidental git-repo-nested Obsidian vault is the same underlying failure mode as the CLAUDE.md/skill duplication bugs from 2026-07-01: content silently existing somewhere it shouldn't, discovered by accident rather than by design.

---

## 2026-07-01 — Periodization + 1RM assignment-time check + inline assign grid + solo RLS fix (app-programs v1→4, calendar-goals v1→2, workouts v3→5, runner v2→3) — PUSHED (76cb53f)

**Done:**
- **Periodization (Linear/Undulating)** — `program_phases.periodization_type`/`periodization_config`, `program_phase_workouts.week_number`/`tier`, `client_program_workouts.week_number` (additive migration). Phase Configure modal (Linear: start/end %1RM + optional deload week; Undulating: Heavy/Moderate/Light tiers per day-slot). `generatePhasePeriodization()` clones Week 1's templates into weeks 2..N with recalculated %1RM only on tagged sets — reps/rest/tempo untouched. Idempotent regeneration + propagates to already-assigned clients + prunes orphaned weeks when duration_weeks is edited down (all via shared `_cleanupPhaseWeeksBeyond` helper).
- **1RM assignment-time check** — `_getProgramOneRMStatus`, shared `_renderOneRMQuickEntry` component (direct kg or Epley estimate, reuses existing `_epley1RM()`), wired into both assign entry points (client-profile + program-page, including solo). Never blocks; PT can quick-fill known/estimated 1RMs inline before confirming.
- **Inline assign-workout grid** — replaced the old "+ Assign workout" modal (day → session → template, one slot at a time) with an always-visible searchable 7-day grid on the phase card; picking a template assigns immediately. `workout_templates.generated_from_phase_id` column added so the picker can exclude periodization-generated clones and client-owned clones. Built specifically to cut modal-reopen repetition, not just polish the old picker — this is the first feature built under the new standing principle "efficiency is the whole platform's spec" (see `project_coachapp.md`).
- 7 new/updated Playwright tests in `tests/programs.spec.js` (periodization Linear/Undulating, 1RM missing/have checklist, inline grid render+search, race-guard, create-workout back-link) — suite now 47 tests.
- Fixed the preview server's `launch.json` in this worktree — stale path (`C:/Users/jaken/coachapp`, doesn't exist) meant the preview had been serving nothing all session until caught. Recurred again mid-session after a process restart auto-started the wrong config ("PT Dashboard" instead of "CoachApp") — same root cause, second occurrence.
- **Correction to an earlier claim in this same log entry:** the 4-piece 1RM plan (runner prompt, Epley estimator, Big 5 quick-start, post-session suggestion) was reported as "already built and live" — that was wrong. `git diff` at push time showed all of it as uncommitted, zero prior git history (last commit touching those files was 2026-06-30). It was written at some point but never committed or deployed. This session's push is the first time any of it actually went live.
- Ran a full quality/maintainability audit on the `hello-claude` skill (11 findings — 4 High, 5 Medium, 2 Low, all fixed): resolved 2 contradictions in Step 1 (blocking vs. non-blocking logic, tool-call ordering), fixed 3 stale `app.js` references, deduplicated the "Mandatory Execution Gates" section against "Standing session behaviours" (was ~100% restated — and one rule had genuinely inconsistent wording between the two copies), grouped 29 flat rules into 5 themed sub-sections, moved "Who Jake is" to the top of the file, documented an undocumented exception to the hardcoded-email rule, clarified an approval-gate ambiguity in the OS self-check.
- Ran the same audit on the `save` skill (6 findings, all fixed) — see the duplicate-skill discovery below for why the first attempt at this didn't actually land.

**Bugs found + fixed:**
- Client calendar (`app-calendar-goals.js`) and client Workouts page (`app-workouts.js`) both assumed one workout repeats identically every week of a phase — periodization's per-week rows would have duplicated/misplaced sessions. Fixed both to be `week_number`-aware while preserving legacy (non-periodized) behaviour exactly.
- `generatePhasePeriodization` never synced already-assigned clients — a client assigned before generation would see weeks 2+ as "Not set up" forever. Fixed: propagates new weeks to every existing assignment.
- Editing a phase's `duration_weeks` down didn't clean up now out-of-range generated weeks (master or client copies). Fixed with `_cleanupPhaseWeeksBeyond`.
- Modal-removal-before-read ordering bug: both assign save functions removed the modal *before* reading the entered 1RM values, so nothing ever saved. Fixed the ordering.
- Race condition: changing the client/program selection then clicking Save fast enough could read stale missing-1RM state, silently dropping entered values. Fixed with a request-token guard (`_oneRMRefreshToken`).
- `_savePostSessionOneRM` and the new `_saveMissingOneRMEntries` both showed a false "saved!" toast even when the `client_1rms` insert failed. Both now check the error and surface a failure toast.
- **Solo accounts had zero write policy for `client_1rms` (insert/update/delete) and no update/delete policy for `client_programs`** — confirmed live with an explicit `42501 RLS violation`. This had silently broken 5 solo features (Add 1RM modal, Big 5 quick-start, runner prompt, post-session suggestion, today's assignment-check) plus "Remove program"/"Edit start date" for solo. Root cause: when solo accounts were built, write policies were added for assignment but never extended to 1RMs or to program removal/editing. Fixed with 5 new RLS policies matching the existing `client_program_workouts` solo pattern; verified all 5 operations live.
- Race condition in the new inline grid's assign function — `session_order` was computed from stale render-time state, so two fast picks (or two tabs) for the same day/slot could both compute the same slot and silently duplicate. Found by a review agent citing concrete evidence (`scripts/fix_session_order.cjs` exists specifically to repair this exact class of collision). Fixed by re-checking the slot is free immediately before inserting, matching the old modal's safety net.
- "Create new workout" from the inline grid created the template but stranded the coach with no path back to the phase they started from — `phaseId`/`dayOfWeek` were captured in context but never consumed. Fixed by wiring in the existing `backFn`/`backLabel` template-editor navigation pattern.
- Preview rendering pipeline occasionally showed a stale wide-desktop paint despite the underlying DOM/CSS being correctly mobile-sized (confirmed via `window.innerWidth`, `scrollWidth`, `matchMedia`) — cycling the viewport resize forced a clean repaint. Not a CoachApp bug.
- `save/SKILL.md` had two stale references from before the app.js modularisation — wrong OneDrive path, and described a single `app.js?v=N` instead of the 8 module files. Fixed.
- **Discovered a systemic problem: 8 CoachApp skills each had a second, independently-drifting copy** — one at the global `C:\Users\jaken\.claude\skills\` path, one at the project's own `C:\Users\jaken\OneDrive\coachapp\.claude\skills\` path. Found by accident: fixed a stale path in `save/SKILL.md`, then re-audited it later and found the same bug still present — the original fix had landed on the global copy, not the project copy being audited. Confirmed empirically (fresh session, watched which copy's distinguishing behaviour actually fired — the preview-verify/launch.json check, which only exists in the global `hello-claude`) that bare skill-name triggers resolve to the **global** copy. Consolidated all 8 (hello-claude, save, sql-safety, deploy-check, mobile-check, security-audit, playwright, run-coachapp) to one golden path at global: merged whichever side had unique/correct content first (sql-safety's project copy had an entire RLS-audit section global was missing; playwright's project copy claimed "3 workers," checked against the real `playwright.config.js` and found it factually wrong — actual config is `workers: 1`, deliberately, to avoid Supabase test contention; 3 of the 8 were already byte-identical), deleted all 8 project-level duplicates, repointed `hello-claude`'s own 8 internal hardcoded skill references from project paths to global paths.

**Decided:**
- 1RM exercise-name matching (fuzzy string vs. moving `client_1rms` to `exercise_id`-linked entry) is a genuinely big design decision — deferred to its own scoping session. Today's build keeps the name-matching logic in one shared helper so swapping it in later is contained, not a rebuild.
- Week 1 alone is sufficient to detect which exercises a program needs (for both periodization generation and the 1RM check) — generated weeks always reuse the same exercise names, just different %1RM values.
- New standing product principle banked: efficiency/time-saving is the whole platform's spec, not a per-feature nicety — when reviewing any builder-type flow, check how many separate trips a common task takes and whether that can collapse to one continuous pass, not just whether each step feels nicer.
- All CoachApp skills live at the global `.claude/skills/` path only, never duplicated at the project level — banked as a rule in `project_coachapp.md` so a new skill doesn't accidentally get created in both places again.

**Open (banked to STATUS.md to-dos):**
- `deleteProgram()` doesn't clean up cloned workout_templates when a program is deleted — found via leftover E2E test debris, not yet a live bug on Jake's account.
- Pre-push hook printed a cosmetic BOM/shebang warning (`scripts/checks.sh: line 1: ﻿#!/bin/sh: No such file or directory`) before running — didn't block the push, all checks still ran and passed after it, but worth a clean fix.
- The 8 project-level skill-file deletions are sitting uncommitted in the CoachApp repo's working tree — asked Jake whether to commit before doing so, since it's repo history, not a local-only cleanup.

**Why:**
- The propagation gap, the duration-shrink gap, the inline-grid race condition, and the create-workout stranding were all caught by the mandatory multi-agent review before push, not by Jake or in production — exactly what that gate is for.
- The skill-duplication bug is the same *class* of problem as the app.js/launch.json stale-path bugs found earlier this session — content silently existing in two places that can drift apart independently. Worth watching for as a pattern, not just today's specific instance.
- The solo RLS gap was found by accident while cleaning up test data, not by review agents or code inspection — a reminder that live testing catches a different class of bug than code review does, and that "an INSERT policy exists" doesn't mean the other 3 commands do.
- The solo RLS gap was found by accident while cleaning up test data (a stuck row that couldn't be deleted), not by the review agents — worth remembering that live testing catches a different class of bug than code review does.

---

## 2026-06-30 — Runner template bug fixes (app-workouts v=2→3)

**Done:**
- `startWorkoutRunner` now fetches the template by ID directly when called from a program Start button, instead of scanning the full coach template list. Fixes blank exercise names and missing set targets for any template beyond position 200 alphabetically.
- Runner setup modal dropdown query raised to `.limit(2000)` — was silently truncating at PostgREST default max_rows=200.

**Bugs found + fixed:**
- Root cause: PostgREST `max_rows=200` cap on both the runner template list query and the setup modal dropdown. Templates beyond position 200 alphabetically returned empty results. Fix: ID-scoped fetch for runner start; explicit `.limit(2000)` for the modal list.

---

## 2026-06-30 — Codebase modularisation: app.js → 8 modules (no version bump)

**Done:**
- app.js (7,968 lines) split into 8 module files: app-core.js, app-dashboard.js, app-programs.js, app-clients.js, app-calendar-goals.js, app-workouts.js, app-runner.js, app-progress.js; loaded in order via 8 script tags in index.html
- Pre-push hook (`scripts/checks.sh`) updated — was hardcoded to `js/app.js`; now scans all 8 module files; cache bust check updated to verify all 8 script tags have `?v=`
- Preview server path fixed in both `.claude/launch.json` files — was pointing to `C:/Users/jaken/coachapp`, now correctly `C:/Users/jaken/OneDrive/coachapp`; this bug was masked by an old copy of app.js and had existed since project start
- `.gitignore` updated — `desktop.ini` and `*.pdf` added; removed from VS Code Source Control
- 40/40 Playwright green post-split; 19/19 smoke tests green on push

**Bugs found + fixed:**
- Preview server 404: launch.json had stale path `C:/Users/jaken/coachapp` (project is in OneDrive). Fixed in worktree launch.json and CoachApp launch.json.
- checks.sh BOM encoding: PowerShell wrote UTF-8 with BOM, causing `﻿#!/bin/sh` warning on each push. Cosmetic only — all checks pass. Fix encoding next session.

**Decided:**
- No rewrite to Next.js — continuing vanilla JS to beta; developer can rewrite frontend against existing Supabase schema post-beta
- Modularisation chosen over full rewrite: same globals, no ES module refactor needed, one session to complete
- 1RM build held for next session (plan already banked in STATUS.md)

**Why:**
- Session felt sluggish with graphify — 7,968-line single file was the root cause; split reduces per-session context load significantly
- No version bump: split is a pure refactor, zero feature changes

---

## 2026-06-29 — Tooling session: iOS fix, graphify knowledge graph, .claudeignore (v179–v180)

**Done:**
- iOS Safari session detail slide-in fix (v180) — replaced `inset:0` with explicit `top:0;right:0;bottom:0;left:0` on both the panel wrapper and backdrop div in `openSessionDetail()`; pushed, CI green
- `.claudeignore` created — excludes node_modules, test-results, lock files, .git, scripts from AI context search
- `graphify` installed and full semantic knowledge graph built — 298 nodes, 545 edges, 26 named communities (App Core, Workout Runner, Goals, Calendar, etc.); wired into Claude Code via CLAUDE.md + PreToolUse hooks; costs $0.15 one-time, future updates are free AST-only
- 1RM build plan approved and banked in STATUS.md for next session

**UNVERIFIED (banked):**
- iOS Safari slide-in fix — confirmed via code review + Playwright (Chromium), not tested on real iPhone

**Decided:**
- Session approach going forward: "Nothing in isolation, everything looking at the big picture" — every feature filtered through product principles before building
- Product principles: (1) Friction at point of use, (2) All features no hidden fees, (3) Copy-paste simplicity — ship with defaults not blank canvas
- Rotate Anthropic API key after use in chat — Jake confirmed deleted

**Why:**
- graphify reduces context burn per session — `graphify query` instead of 10 grep passes through 7k-line app.js; article cited 40-60% context reduction from .claudeignore alone

---

## 2026-06-29 — Supabase security hardening pass (no version bump)

**Done:**
- OpenAPI schema mocked via `public.mock_openapi()` — schema structure no longer publicly readable via anon key
- `public.lock_created_at()` trigger function created; applied as BEFORE UPDATE trigger on 11 core tables (workout_logs, weight_logs, performance_logs, goals, goal_check_ins, goal_milestones, events, client_check_ins, client_1rms, sessions, logged_sessions) — created_at is now immutable
- `delete_current_user` search_path tightened from `search_path=public,auth` to `search_path=''`
- `events` UPDATE policy ("Client manages own personal events") fixed — USING clause was `true` (open), now scoped to `EXISTS (clients where user_id = auth.uid()) AND is_pt_assigned = false`
- REVOKE EXECUTE on `handle_new_user`, `lock_created_at`, `log_audit_event`, `mock_openapi` from both anon and authenticated — these are internal/trigger functions that should never be callable via API
- REVOKE EXECUTE on `delete_current_user` from anon only (authenticated keeps access — that's the delete account flow)
- Max rows: 1000 → 200 (Data API settings)
- Auto-expose new tables: toggled off (new tables no longer get API access automatically)
- Secure password change: enabled (requires recent login to change password)
- Minimum password length: 6 → 8
- Security Advisor result: 0 errors; remaining warnings are Pro-plan features or intentional patterns

**Decided:**
- Supabase book ("Building Production-Grade Web Applications with Supabase", Packt 2024) fully assessed; 5 priority actions all executed this session
- OpenAPI schema exposure is a known Supabase default — now mitigated
- Leaked password protection skipped — Pro plan only, not available on Free tier

**Why:**
- Events UPDATE USING clause was a real gap — any authenticated user could select any event row for updating (WITH_CHECK was correct but USING was open)
- Internal trigger functions were callable via REST API — revoking EXECUTE removes them from the public API surface without breaking their trigger behaviour

## 2026-06-29 — Session detail slide-in bug fix + security hardening (v176–v179)

**Done:**
- v176: `openSessionDetail(templateId, name)` slide-in built for solo/client workouts page. Fetches `workout_template_exercises` and renders a right-side drawer. 39/39 Playwright green.
- v177: Fixed solo 1RM library not loading (was gated by `isClientPlan`); added propagation toast when shared templates are edited (warns PT that changes affect all assigned clients).
- v178: Re-fixed session detail slide-in not appearing on live site. Root cause: panel wrapper was unstyled div — `position:fixed` children failed to layer above `overflow:hidden` app shell. Fix: panel itself set to `position:fixed;inset:0;z-index:1000`; children changed to `position:absolute`. Matches `.modal-overlay` pattern.
- v179: `sudoAsClient()` in-function email guard added (`if (currentUser?.email !== 'jakendwest@gmail.com') return`) — was callable from DevTools by any authenticated user. Session detail slide-in smoke test added to `tests/solo-account.spec.js` (19 smoke tests).
- bf3bac7: Smoke test `hasPhase` guard added — skips session detail test gracefully when E2E solo client has no program assigned.

**Bugs found + fixed:**
- Slide-in not appearing: unstyled container div + `overflow:hidden` on `.app-shell` caused stacking failure. Fixed by making wrapper `position:fixed;inset:0;z-index:1000` (same pattern as every other modal).
- `sudoAsClient()` security gap: function was only gated in the UI render; any logged-in user could call it from DevTools. Fixed with in-function email check.
- Smoke test timing out: new test waited for `button[onclick*="cl-phase"]` which never appears on E2E account (no program assigned). Fixed with `hasPhase` guard and early return.
- STATUS.md + LOG.md not updated: /save was not run at end of prior sessions — STATUS.md was 4 versions stale (showed v175, was at v179). Fixed this session.
- CRITICAL.md showed privacy policy as ❌ — it was built in v158 and is live. Fixed.

**Decided:**
- `openSessionDetail` relying entirely on Supabase RLS for access control is acceptable — no JS-level auth check needed because the DB query itself is scoped.
- 8 standing session behaviours promoted to MANDATORY EXECUTION GATES in `hello-claude/SKILL.md` with explicit trigger→action pairs — no longer guidance, now hard stops.

**Why:**
- Slide-in stacking: `position:fixed` inside a flex child that has `overflow:hidden` can cause the fixed child to be clipped in Chrome — the outer `overflow:hidden` creates a new stacking context. Making the wrapper itself `position:fixed` bypasses this.
- Mandatory gates: standing behaviours were being systematically skipped because they were listed as guidance. Jake asked for them to become enforcements after identifying 8 were missed this session.

---

## 2026-06-29 — Dashboard rework + sudo mode + activity feed fix (v173–v175)

**Done:**
- v173: All three dashboards (PT, client, solo) restructured — hero "Up next" card (current program + phase + week), stats strip (hidden on mobile for solo), two-column grid collapsing to single column on mobile. Phase name redundancy fixed (`/week/i` guard so "Base Building · Week 2" doesn't become "Base Building · Week 2 · Week 2"). Progress tabs: `+ Log weight` and `+ Log PB` add buttons added to Body Weight and Personal Bests tabs. Cardio empty state explains auto-population.
- v174: Sudo/impersonation mode — `sudoAsClient(clientId, clientName)` and `exitSudo()` functions. "View as" button on each client list row (email-gated to jakendwest@gmail.com). Renders full client dashboard using PT's RLS access. Amber banner "👁 Viewing as [Name]" with "Exit ✕" button restores PT view and clears sudo state.
- v175: Fixed PT dashboard activity feed showing solo sessions as "Unknown" — pre-fetched coach's client IDs (`coachClientIds`) and scoped `weight_logs` + `workout_logs` queries with `.in('client_id', coachClientIds)`. "Sessions this week" stat now correctly counts only client sessions (was counting Jake's solo sessions).

**Bugs found + fixed:**
- PT activity feed "Unknown" entries: `workout_logs` + `weight_logs` queries had no `coach_id` or client scope — Jake's solo sessions appeared in PT feed labelled "Unknown". Root cause: unscoped multi-tenant queries, caught by hello-claude code review. Fixed by pre-fetching client IDs and using `.in()` filter on both queries.
- Hero meta redundancy: phase named "Week 2 — Foundation" was getting "· Week 2" appended. Fixed with `/week/i.test(currentPhase.name)` guard.

**Decided:**
- Sudo mode is email-gated (not role-gated) — it's a dev tool for Jake only, not a feature real PTs would have
- Mobile stats strip hidden on solo dashboard; PT stats strip stays visible at all widths (3-column grid fits at 480px)

**Why:**
- Dashboard rework: mixed SaaS (clean two-column data layout) + fitness app (hero card) approach chosen over pure SaaS after mockup comparison
- Sudo: PT already has RLS access to all client data, so impersonation just means rendering the client view with a specific clientId bypassing the user_id lookup — no auth changes needed

---

## 2026-06-29 — Personal (solo) account build + process self-audit (v158–v162)

**Done:**
- v158: Personal account full build — PT | Personal three-pill view switcher (desktop + mobile), `window._soloClientId` global, `loadUserInfo` detects solo vs coached client records separately, `_getCurrentClientId()` helper routes to correct record per role, `renderSoloDashboard()` new function (stats, recent sessions, weight, upcoming, goals, PBs), solo nav (no Clients), `_NAV_ITEMS.solo`, `showAssignProgramModal` auto-assigns in solo mode, `renderProgressWeight/Strength/Cardio/PBs` all fixed to use helper instead of bare user_id query, `renderProgressWeight` bug fixed (was querying weight_logs by user_id — column is client_id), privacy-policy.html created (UK GDPR compliant, 11 sections), consent checkbox href updated
- SQL: `UPDATE clients SET coach_id = null` on Jake's client record — severs it from PT account, becomes personal record; 4 new RLS policies on `client_programs` (INSERT + SELECT) and `client_program_workouts` (INSERT + SELECT) for solo users
- v159: Fixed `ReferenceError: currentView is not defined` in `renderCalendar` — broke calendar for ALL roles
- v160: Programs page in solo mode — "Add to my plan" button label, `showAssignProgramToClientModal` skips client picker in solo mode, `saveAssignProgramToClient` accepts pre-filled solo clientId, empty state copy updated
- v161: Start button on template detail for solo mode (and client context); sql-safety skill new "new role audit" section; hello-claude golden path walk standing behaviour added
- v162: `renderWorkouts` routes solo to `renderClientWorkoutsPage` (program session accordion with Start buttons); `renderClientWorkoutsPage` falls back to `currentUser.id` when `coach_id` is null; `renderWorkoutTemplates` adds `.is('client_id', null)` to exclude cloned plan templates from flat list

**Bugs found + fixed:**
- `currentView` not defined: stale variable from old master account pattern referenced in `renderCalendar` isClient check — broke calendar for all roles. Root cause: blast-radius sweep didn't catch that shared functions need all-roles verification
- RLS 403 on `client_programs` INSERT: existing policy only covers coach-scoped inserts; solo client record has `coach_id = null` so was rejected. Root cause: new role audit not performed before testing. Fixed with 4 new policies
- Cloned program templates appearing in PT Workouts list: `_cloneProgramForClient` creates templates with `client_id` set and `program_id = null`; `renderWorkoutTemplates` filtered by `!program_id` but not `!client_id`. Fixed with `.is('client_id', null)` in query
- Solo Workouts page showed template builder instead of session accordion: `renderWorkouts` only routed `role === 'client'` to `renderClientWorkoutsPage`; solo users need the same view to start sessions. Root cause: user journey not walked end-to-end before declaring done
- `renderClientWorkoutsPage` templates query `.eq('coach_id', clientRecord.coach_id)` — returns nothing for solo (coach_id = null). Fixed with `|| currentUser.id` fallback

**Decided:**
- Personal account architecture: one client record per user (unique constraint on user_id), so "personal" = existing record with coach_id nulled, not a new cloned record
- Solo user sees client-style Workouts (program accordion) + PT-style Programs (builder). Workouts = execute, Programs = build
- Process self-audit: identified 5 isolation gaps in review process — all-roles check, data flow check, post-build review trigger, golden path walk, RLS role audit. All added as standing behaviours

**Why:**
- Unique constraint on clients.user_id prevented the planned "clone to new record" approach; nulling coach_id is architecturally cleaner anyway — no data duplication, same RLS anchor

---

## 2026-06-29 — Delete modal fix + settings smoke tests + smoke test standing behaviour (v155→v157)

**Done:**
- v156: Delete account modal position fix — `align-items:flex-start;padding-top:60px` on overlay so modal anchors near top of viewport regardless of scroll position; previously the modal was vertically centred and the top half was clipped above the visible area when triggered from the bottom of the Settings page
- v157: `tests/settings.spec.js` — 5 new smoke tests: settings sections render, delete modal opens, cancel closes modal (`waitForSelector detached`), validation error shows without correct input, download button is clickable. Suite now 31 tests.

**Bugs found + fixed:**
- Delete modal clipped at top of screen: `position:fixed` + `align-items:center` centres in viewport, but Delete button is at the bottom of a long scrollable page — top of modal was above visible area and user couldn't scroll to reach it. Root cause: regression sweep didn't ask "what happens when this modal is triggered from the bottom of a scrolled page." Fixed with `flex-start` + `padding-top:60px`.

**Decided:**
- Smoke tests added for every new feature in the same commit going forward — modal opens, form renders, button responds. Deeper data-writing tests discussed first. Jake approved this pattern.
- `waitForSelector('.modal-overlay', { state: 'detached' })` is the correct pattern for asserting modal removal (more reliable than `not.toBeVisible()`)
- `/code-review ultra` is not available in Jake's desktop app — local multi-agent review (3 finders + verifier) is the equivalent; never prompt Jake to run the slash command

**Why:**
- Live smoke test caught the modal clipping that Playwright and the regression sweep both missed. Playwright doesn't check visual clipping; regression sweep didn't consider scroll context. Smoke test now covers this class of issue.

---

## 2026-06-29 — OS self-audit continuation + code review + XSS fix + push (v152→v155)

**Done:**
- v153: Code audit fixes — `removeBrandingLogo` fire-and-forget DB write now error-handled; dead code block removed from `renderClientPhotos`; `openClientByName` (orphaned, unscoped query) deleted
- v154: `deleteAccount` replaced `confirm()`+`prompt()` with custom modal — type DELETE to confirm, inline validation error, button disabled during deletion
- v155: XSS fix — added `escapeHtml()` helper; applied to all `businessName` innerHTML injection points (sidebar, client dashboard, PT dashboard subtitle, settings input value); wrapped `downloadMyData` in try/catch so network errors show a message instead of leaving UI stuck
- All 11 commits pushed to master; GitHub Pages deployed; CI green (both workflows passed)
- Open RLS policy fixed: `workout_templates` "Client reads workout templates" was `qual = true` (any client could read all coaches' templates); rewritten to scope to `coach_id = (SELECT coach_id FROM clients WHERE user_id = auth.uid())`

**Bugs found + fixed:**
- XSS (HIGH): `coach_branding.business_name` injected raw into `innerHTML` visible to clients — a coach could execute script in clients' sessions. Fixed with `escapeHtml()` at all 5 injection points
- `downloadMyData`: no try/catch — `Promise.all` rejection left UI permanently showing "Preparing download…". Fixed with try/catch + user-visible error message
- `removeBrandingLogo`: storage.remove() was error-handled but the subsequent db.update() was not — inconsistent state if update failed. Fixed
- `renderClientPhotos`: duplicate unreachable empty-state check. Removed
- `openClientByName`: orphaned function never called, contained unscoped `clients` query (no coach_id filter). Deleted
- Playwright test timeout 10s too tight under full-suite load after `_loadBranding()` added extra login round-trip. Bumped to 20s — 26/26 stable
- Open RLS policy on `workout_templates` — clients could read any coach's templates. Fixed in Supabase

**OS audit completed:**
- Both `hello-claude` files (account + CoachApp) were reading PTHub paths, missing 4 standing behaviours, saying "Seven checks" (now Nine)
- `run-coachapp` and `mobile-check` at both levels still referenced Netlify
- Playwright skills said "X/14" — suite is 26 tests
- 7 skills were CoachApp-level only (not account-level) — added: deploy-check, save, security-audit, sql-safety, mobile-check, playwright, run-coachapp
- `/code-review ultra` confirmed not available in Jake's desktop app — behaviour updated: run local multi-agent review inline before every push
- LOG.md 2026-06-28 entry written (entire prior session had no log record)
- Roadmap corrected: Branding marked Done in both rows

**Decided:**
- `escapeHtml()` is now the mandatory pattern for any coach-controlled string in innerHTML
- Pre-deploy code review runs as inline multi-agent (3 finders + verifier), not as a slash command

**UNVERIFIED (banked):**
- Live smoke test on GitHub Pages post-push (branding, GDPR features, delete account modal) — not tested in browser on live URL

---

## 2026-06-28 — Branding + security/GDPR hardening + OS self-audit (v134→v152)

**Done:**
- v150: Edit sessions from Programs page; exercise library dropdown in edit modal; `program_id` clone bug fix; timed set render fix (1:30 not 90 reps); PII stripped from 16 log sites; pre-push checks expanded to 10
- v151: PT branding — `coach_branding` table, private `logos` bucket, signed URL (604800s), sidebar + PT dashboard + client dashboard display
- v152: Security/GDPR hardening — `progress-photos` bucket made private, signed URLs for photos (3600s), consent checkbox on signup, Data & privacy card in Settings (data export + delete account), `delete_current_user()` Postgres RPC

**Bugs found + fixed:**
- `hello-claude` skill reading `PTHub\STATUS.md` instead of `CoachApp\STATUS.md` — every session started with wrong project context
- `/save` skill writing to `PTHub\STATUS.md` and `PTHub\LOG.md` — every session save went to the wrong project; STATUS.md was 18 versions stale
- `alert()` in `deleteAccount()` blocked by pre-push hook — replaced with inline DOM message
- `progress-photos` bucket was public — made private, migrated display from `getPublicUrl` to `createSignedUrls` batch

**Decided:**
- Signed URL expiry standards: logos = 604800s (7 days), progress-photos = 3600s (1hr)
- `delete_current_user()` as `security definer` Postgres RPC — avoids needing service role key on client side
- Double-confirm for account deletion: `confirm()` + `prompt()` requiring "DELETE" typed verbatim
- Pre-push hook must use inline DOM message, never `alert()` (blocked by hook check 6)

**Skills created/updated:**
- `/security-audit` — new skill; 7-point checklist covering buckets, RLS, PII, GDPR, new tables, signed URLs
- `deploy-check` — expanded to 9 checks (added 5b: buckets private, 5c: GDPR features)
- `hello-claude` (both account-level and CoachApp-level) — fixed PTHub paths, added 4 security gates, sounding board, approve-before-build, regression sweep, blast radius sweep
- `/save` — fixed PTHub paths in Steps 3 and 4

**Memory created:**
- `feedback_security_gdpr.md` — comprehensive security/GDPR rules
- `project_coachapp_patterns.md` — modal pattern, program_id constraint, timed sets format, dbq(), master account, nav context, save functions

**UNVERIFIED (banked):**
- Branding display in preview (confirmed in session), but not tested on live GitHub Pages
- `delete_current_user()` RPC — runs in DB but full end-to-end deletion not tested with a real account

**Why:**
- Security was missed because no proactive gates existed, lessons stayed in volatile STATUS.md not permanent memory, and skills were never audited after creation. This session adds all three missing layers.

---

## 2026-06-27 — Runner feature pack + regression fixes + Playwright hardening (v134)

### What was done
- **Timed sets:** Template builder shows Duration (mm:ss) field when Timed toggle active. Runner shows duration log field and validates before logging. `flushTemplateSets` already handled the field; runner `logRunnerSet()` checks `tgt.timed` first.
- **Unilateral L/R logging:** When a set's `sets_json` has `unilateral: true`, runner shows separate Left/Right columns (weight + reps each). `setData` carries `leftWeight/leftReps/rightWeight/rightReps`. Frontend-only change, no schema change.
- **Client "My Programs" accordion:** `renderClientWorkoutsPage()` now queries `client_programs` with full phase/workout structure and `workout_template_exercises`. Shows phases that expand to session rows (exercise count + day). Start button on right. Falls back to flat template list if no program assigned.
- **Runner UI redesign:** (1) Set X of Y in header, (2) reps/weight target visible below exercise name, (3) PT notes always shown with `[Label]` prefix stripped, (4) per-exercise client notes textarea stored on `_runner.exercises[i].clientNotes`, (5) visible "✎ Edit" button on logged sets, (6) scrollable logged-sets area.
- **Skip rest bug fixed:** `skipRestTimer()` was clearing the rest overlay but never calling `renderRunner()` for between-sets rests (where `_afterRest` is null). Fix: `else if (_runner) renderRunner()` added at end of function.
- **Client session detail regression fixed:** New `renderClientWorkoutsPage` had removed the old expandable session detail. Fixed by: adding `workout_template_exercises` to the query, making session name tap-to-expand via `toggleClientPhase(detailId)`, rendering exercise list in hidden panel.
- **Client notes schema:** `workout_log_exercises.client_notes text` — SQL migration written; Jake must run it manually.
- **Playwright tests strengthened:**
  - `skip rest clears rest overlay and restores input fields` — now asserts: overlay gone, "Resting" text absent, LOG button visible. Would have caught the skip rest bug.
  - `client can tap session name to see exercise list` — asserts exercise detail panel opens on tap. Would have caught the session detail regression.
  - Fixed stale `text=START A WORKOUT` assertion (renamed to `h1` containing "Workouts").
  - `beforeEach` in runner tests now expands first phase panel if accordion is present, ensuring Start buttons are visible.
  - All 15 tests green.

### Decisions
- **client_notes stored per exercise, not per set** — logging context (feeling, cue) is exercise-level. Set-level is covered by weight/reps data. Per-exercise textarea is simpler and sufficient for v1.
- **Accordion tests are conditional** — if test client has no program assigned, accordion tests skip gracefully. Consequence: these tests are currently no-ops for the Playwright test client. Fixing this requires assigning a program to the test client, which is a manual step.
- **Timed/unilateral are frontend-only changes** — no schema changes needed. Both flags live in `sets_json` already. This is the correct architectural decision: set metadata belongs in the template's JSON, not as separate columns.

### Revisit-if
- Client notes will silently fail until Jake runs the SQL migration. First symptom: `client_notes` column missing → Supabase returns error → `saveRunnerSession` logs the error but the session saves anyway (partial data).
- Timed sets and unilateral UI are unverified on live site. Smoke test needed after next push.
- Playwright accordion tests are conditional — if test client gets a program assigned in future, the condition will start running real assertions and may surface bugs.

### Regressions surfaced this session
- **Skip rest**: `skipRestTimer()` had no `renderRunner()` call after clearing between-sets rest. Fixed. Playwright guard added.
- **Client session detail**: Replacing `renderClientWorkoutsPage` removed the expandable session row pattern. Fixed by re-adding expand with `toggleClientPhase`. Playwright guard added.
- **Root cause of both**: Redesigning a page without asking "what did the old code show that the new code now hides?" Both failures would have been caught by the regression-sweep habit. Jake confirmed both could have been prevented — "yes" answer to "could both have been prevented?".

### App version at close
v134 — committed locally; not yet pushed to master.

---

## 2026-06-25 — Runner redesign, role-specific nav, My Progress page (v110)

### What was done
- **Role-specific nav:** Replaced static nav HTML with `renderNav(role)` function. PT gets 6 items; client gets 4 (Dashboard, Workouts, Progress, Settings). `_NAV_ICONS` and `_NAV_ITEMS` objects define each role's nav. `applyRoleUI()` now calls `renderNav()` on login.
- **Compact runner input area:** Replaced 3-column grid layout with single flex row — Set N (left column) | Kg input (flex:1) | Reps input (flex:1) | LOG button + skip/finish. Inputs reduced from `font-size:28px` to `22px`, padding from `10px` to `6px`. 
- **Last session strip:** Added persistent `#wr-last-session` div above the stats bar in runner (for non-cardio exercises). `fetchRunnerLastSession(exName)` uses a two-query pattern (logs by client_id → exercises by log_id). Shows "Last · 24 Jun  S1 80kg×8  S2 80kg×8" style.
- **Next exercise moved to header subtitle** — removed the "Next up" card from the scrollable set log area; added a one-liner `Next: <exercise name>` below the exercise name in the header.
- **Runner chart fully removed** — `fetchRunnerExHistory` and `drawRunnerChart` deleted. Progress data now lives in My Progress page where it belongs.
- **My Progress page (client):** 4-tab shell with `renderProgressWeight`, `renderProgressStrength`, `renderProgressCardio`, `renderProgressPBs`. `navigate()` updated with `case 'progress'` route. Body Weight tab renders Chart.js weight chart. Strength tab uses `workout_logs!inner` join to filter by client. Cardio and PBs tabs structured but thin.
- **Template edit/delete guard:** Added `.eq('coach_id', currentUser.id).select()` and row-count check to `saveEditTemplate` and `deleteTemplate`. Silent RLS failures now surface as user-visible errors.
- **Sounding board protocol:** Jake asked to be challenged on new feature ideas before building. Memory saved; now applies to all new feature requests this session onward.
- **Pushed to production:** d1d4377 → master → GitHub Pages.

### Decisions
- **My Progress page is the home for all historic client data** — not the runner, not the client dashboard. Charts in the runner were distracting during a workout; progress review belongs in a dedicated page. Jake proposed this; confirmed he wanted Body Weight / Strength / Cardio / PBs tabs.
- **Role-specific nav solves the overflow problem cleanly** — rather than hiding items, each role gets a purpose-built nav set. No CSS overrides, no hidden elements.
- **Last session strip is persistent (not in empty state)** — initially rendered in the "no sets yet" empty state; Jake asked for it to be above the stats bar persistently so it's always visible during the set.

### Revisit-if
- My Progress Strength tab uses PostgREST `workout_logs!inner` join with `.eq('workout_logs.client_id', ...)`. If this returns no data on production, the filter syntax may need to be `.eq('workout_logs.client_id', ...)` → PostgREST embedded filter format. Check Supabase API logs after first live test.
- `performance_logs` RLS for clients: current policies may not cover this table for client SELECT. Check if Personal Bests tab errors on live.

### App version at close
v110 — pushed and deployed to GitHub Pages.

---

## 2026-06-25 — Programs feature + modal layout fix

### What was done
- **Programs full build (v94):** Phases UI already existed; added assign-template-to-day workflow, "Assign to client" button + modal on program builder, weekly schedule card on client dashboard with today highlighted + ▶ Start button that launches the workout runner with the correct template. Verified end-to-end: Jake's program (12-Week Strength Block) assigned to Jake West client, dashboard showed Mon/Wed/Thu workouts, Start button launched runner.
- **Critical layout bug fixed (v95):** Program builder page showed two copies of page side-by-side on desktop. Root cause: `apc-modal` was embedded as static HTML inside `openProgram`'s `el.innerHTML`. Unstyled because it used `class="modal-box"` (doesn't exist in main.css; correct class is `modal`). Fixed by converting to dynamic body-level modal creation (matching `showAssignProgramModal` pattern). Also fixed `modal-box` → `modal` in `openProgram` and `renderPrograms` modals.
- **Preview resize confusion:** Resized preview to 1200px for desktop test, forgot to resize back. Jake reported "two pages still rendering" — was the normal sidebar + main content desktop layout, not a rendering bug. Resized back to 480×844.

### Decisions
- **Dynamic modal pattern is the standard:** Any modal must be created via `document.createElement` + appended to `document.body`, never as static HTML in `el.innerHTML`. Reason: static modals inside `#main-content` (which has `overflow-y:auto`) interfere with fixed positioning. Pattern source: `showAssignProgramModal` line ~942.
- **`modal` not `modal-box`:** `.modal-box` does not exist in main.css. All modal inner containers must use `class="modal"`. Reason: wrong class = no CSS = unstyled overlay that breaks layout.

### Revisit-if
- Desktop layout at >768px viewport needs a full UX review pass — cards may not stretch to full main-content width. Not confirmed yet.
- Assign-to-client modal not verified on live site — only in preview.

### App version at close
v95 — pushed and deployed to GitHub Pages.
