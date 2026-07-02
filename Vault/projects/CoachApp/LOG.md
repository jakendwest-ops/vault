# CoachApp — Session Log

Newest first.

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
