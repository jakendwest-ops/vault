# CoachApp Roadmap

_Last updated: 2026-07-23 (2nd save — exercise-builder overhaul `c72eb14` + day-row prescriptions & picker disambiguation `b53dbfc`, both LIVE; ④ coach parity still remains)_

---

## 🐛 Session backlog — 2026-07-23 (exercise-builder overhaul, then day-row prescriptions + picker)

_Two pushes. Jake opened with a UX critique rather than a bug report ("very very similar to the heavyset exercise builder and I dont want to get pulled up on being a copycat"; cardio wrong; jumps unfinished; too much scrolling), then followed with four more UX issues from live use. Both rounds shipped; the Library page was deliberately left for its own scoping session._

**✅ Shipped `c72eb14` — exercise-builder overhaul (app-workouts v31, app-runner v29, app-progress v21, app-programs v21, main.css v6):**
- Cardio distance in **metres** (new `distanceM` key + shared `_cardioDistanceM`/`fmtDistanceM`; legacy km read-only, never rewritten — fix forward). **Watts** end-to-end (builder targets → runner input → Progress chip; new `avg_watts smallint`, migration run live by Jake, clamped at the save site). `Pace / km` retired to a legacy-only escape hatch. Optional fields behind a `+ More targets` disclosure — **cardio 3 sets at 390px: ~1320px → 685px**; weight_reps 1138px → 832px.
- **Jump targets** finally prescribable (height/distance/jumps/rest/RPE) + the missing `_buildTargetCols` jump branch — the runner target bar had rendered **empty** for jumps.
- **54 undefined CSS vars defined** + builder repainted 15 hardcoded greys → 0 (the copycat half — the builder had no visual identity because its tokens were dead).
- Heavyset teardown ingested into `coachapp-client-app-benchmarks`.

**✅ Shipped `b53dbfc` — day-row prescriptions + picker disambiguation (app-workouts v32, app-programs v22, main.css v7):**
- Day rows show the real prescription (`4 × 8–10 reps · 100kg · RPE 8 · 2:00 rest`) on **all four** surfaces that render that block — the fourth (coach's view of a client's plan) was caught by the review.
- **One shared `_fmtSetDetail`/`_fmtSetsCollapsed`** replaced two silently-drifted copies.
- Picker rows: `↳ Used in <phase> · Wk N · MON` vs `Not used yet`, + exercise count. Two duplicated pool builders collapsed into `_buildProgramTemplatePool`.

**🐛 Bugs found + fixed that Jake never reported:** duration-based cardio silently discarding its distance (`distanceAchieved` was a key the save path never read); legacy `'0:00'` pace being truthy and rendering phantom chips; the ADD path's `cleanSets` allowlist discarding **every** cardio target except duration/distance since those fields shipped (EDIT kept them); `metric_type` dropped by **both** clone paths so every ASSIGNED plan lost its shape routing.

**🔴 Found, NOT fixed — needs its own task:** every "mobile check" this project has ever run was at **1280px, not 390px**. `playwright.config.js:13` sets a mobile viewport but line 19's `devices['Desktop Chrome']` overrides it. The `mobile-check` skill's central claim is therefore false. Fixing it re-baselines all 181 tests.

**🗓 Still open (needs Jake):** the Programs-page add-exercise report — driven live in both cases (same-named siblings → modal fires; differing names → re-render fires) and **neither symptom reproduced**. Needs his actual program.

**🗓 Deliberately deferred:** Library page redesign — Jake: *"this may need a scoping session on its own."*

**Process lessons:** les-047 (a shared helper's parameters are a contract each call site must satisfy — an out-of-scope identifier is a runtime error invisible to every static gate) and les-048 (merging drifted copies destroys exactly the drift that justified deduping — diff the old copies against EACH OTHER, not just against the new one).

---

## 🐛 Session backlog — 2026-07-19 (Progress overhaul — capture layer, branch `progress-overhaul`, NOT pushed)

_Jake (screenshot of his Personal Progress page): "very sparse… does not show as much data as it needs to… a core feature for coaches tracking clients over weeks/months/years" — bodyweight, exercise progressions (sets/reps/weight/volume), cardio (duration/times/effort/HR/resting HR), unilateral/AMRAP/duration, and jump height/distance. Brainstormed end-to-end → spec → 4 sequenced sub-projects. Built the whole **capture layer** this session; the **display** (③) is where it becomes visible._

- **✅ ① Data model** — `metric_type` (6 values) on exercises/template/log rows + typed `avg_hr`/`max_hr`/`height_cm`/`side` on `workout_log_sets`. Migration + supplementary backfill run live by Jake in Supabase. (`scripts/add-metric-type-2026-07-18.sql`, `scripts/backfill-metric-type-flags-2026-07-18.sql`)
- **✅ ②b Save-persistence** — runner save stops dropping unilateral/timed/distance/HR; stamps metric_type. app-runner v24.
- **✅ ②a Builder picker** — 6-option metric_type picker replaces Strength/Cardio; drives set fields; derives legacy fields. app-workouts v30. **Model revised: 6 types, AMRAP stays a per-set flag.**
- **✅ ②c Adaptive fast table** — runner fast table renders columns per metric_type; wizard retired for strength types (cardio only). app-runner v25. 8 stale runner tests updated.
- **✅ ②d Manual HR (SHIPPED)** — cardio avg/max-HR inputs in the runner + resting HR on the **bodyweight form** (moved off check-in — that form is client-dashboard-only, so solo couldn't reach it; Jake's call). `resting_hr` on `weight_logs` (migration `scripts/add-resting-hr-2026-07-19.sql`, run live). app-runner v26, app-clients v7, app-dashboard v5.
- **✅ ③ Display rebuild + SetGraph-informed analytics (SHIPPED)** — app-progress v20, app-runner v28. **B1** metric_type trend cards for every type (top weight/est-1RM/volume, cardio distance-pace-HR, unilateral **L-vs-R** dual line, jump, timed) + **Personal Records** block (best set `100 kg × 10`) + range selector + smart weekly/monthly aggregation. **B2** Intensity (kg/rep). **B3** Recent-sessions **diary** — per-workout summary tiles + per-metric **vs-previous deltas** (▲/▼ + %) + set-details line; Per-exercise now the default sub-tab. **B4** resting-HR trend on Body tab. **B5** Cardio-bests section removed. **B6** our own metric colour palette. **C** live runner **"vs last session"** block (shows from the moment you reach a strength exercise). SetGraph analytics reference ingested into the wiki. _Deferred: the finish-screen volume under-count for unilateral/timed was NOT tackled — still open._
- **🗓 ④ Coach parity** — factor the trend view to `(clientId, role)` and render it read-only in the coach's client-profile too. Today `renderClientPerformance`/`renderClientWeight` still show the OLD view; resting-HR input/chart are self-view only. **The queued fast-follow.**
- **Done:** multi-agent-review (0-blocking, fixed a duplicate `_epley1RM`) → merged to master → **pushed live, CI green** (two deploys: `a02292e` then `95e8e8f`). Full suite 168/1-flaky/2-skip.
- **Decisions locked:** analytics mirror SetGraph's DATA depth, never its design (our flat cards/wording/colours/Chart.js — anti-plagiarism, Jake flagged it twice); the runner vs-last block shows a "beat it" reference from the start (deltas fill in as you log — his live-use call); resting HR on the bodyweight form (solo-reachable); per-workout analytics = the enhanced diary now (session-trend chart is a later fast-follow); avg-rest deferred (needs new per-set timestamp capture). Prior locked: first-class metric_type drives capture+storage+charts; typed columns; one adaptive fast table; unilateral = two side-rows; AMRAP = per-set flag.

**Also (process, this session):** `feedback_paste_sql_inline` — Jake wants runnable SQL pasted inline every time, never a file path. Two implementer subagents broke down mid-task (garbled return / uncommitted / stale report) → controller verified actual git+file state and took over; banked as a lesson. Concurrent Playwright runs against one dev server contaminated a full-suite result (8 false failures) — banked.

---

## 🐛 Session backlog — 2026-07-18 (week-tabs redesign — Programs builder + client Workouts page)

_From Jake's live feedback (screenshots of his Personal account): the Workouts-page workout-detail slider felt redundant and weeks/days looked alike; the Programs builder was unappealing and scrolled sideways on mobile. Brainstormed → approved a unified **week-tabs** model → built both surfaces → multi-agent review → pushed 07febfa. CI green._

- **✅ Done:** unified week-tabs model on both surfaces — weeks = tabs, a day/slot opens its workout inline, the session-detail slider is removed from both, the builder stacks days on mobile (no sideways scroll) / grid on desktop, and builder slots carry Edit / Remove / Save-to-Library inline. app-workouts v29, app-programs v20, main.css v5. New `tests/week-tabs.spec.js` (builder + read-page regressions — the read page had **no** CI coverage before).
- **✅ Fixed in review:** restored the per-workout Save-to-Library button (silently dropped when the slider was removed); escaped a pre-existing raw DAY-header interpolation (coach→client stored-XSS).
- **🗓 Live-verify (Jake):** confirm both surfaces on his own account with his real programs.
- **🆕 Found, deferred (Low):** `navigate('programs')` has no client role re-guard (RLS holds, safe today) — add a role check eventually.
- **Decisions locked:** one shared week-tabs model across read + build; days stack on mobile, multi-column on desktop (builder only); tap a day/slot opens it inline (no slider); Save-to-Library kept as a per-workout action (Jake's call over bulk-only).

**OS (same session):** os-lint gained a `stale-predictions` check (RED on any CoachApp prediction past `verify_by` still `outcome:null`; PTHub excluded), and hello-claude's manual "Step 8 — Predictions" was removed — the hook owns prediction staleness now. Proven RED→GREEN on fixtures; backed to claude-config.

---

## 🐛 Session backlog — 2026-07-17 (OS rebuild + full-file review found 18 latent bugs; pushed 134140f)

**OS/tooling (shipped to `~/.claude`, not app code):** built the approved OS rebuild — `os-lint.mjs` staleness hook (9 checks, silent-when-clean, each proven RED→GREEN on a planted fault), bug-ledger intake+closure rules, hello-claude 33→13 standing behaviours, `/save` PII gate + 9-module cache-bust, PII stripped from skills, `post-build-review`+`security-audit` deleted. os-lint now backed up to claude-config (hooks/+state/ allowlisted).

**App fixes (pushed, `134140f`, all 7 modules bumped — app-core v5, app-clients v6, app-dashboard v4, app-programs v19, app-progress v11, app-runner v23, app-workouts v28):** the first-ever full-file multi-agent review found **18 latent bugs** in code no diff had touched; fixed those + 4 regressions the pre-push review caught. Stored XSS (client→coach, defeats RLS) fully swept codebase-wide; live data-loss chain (stale `_phaseWorkoutContext`); zero-session-phase crash; client had coach's Delete+notes; `deleteProgram`/`deletePhase`/`removePhaseWorkout` debris/orphan fixes (one shared guard, all 5 delete paths); silent-failure fixes; runner teardown/re-entrancy/1RM fixes; solo delete-template eject. 9 red→green tests; suite 151/2/0.

**New this session:**
- **`feedback_solo_null_coach_id` (les-042):** a `coach_id` filter silently excludes the solo record — now 4 bugs of this exact shape. Solo tests skip in CI, so only the pre-push review catches this class.
- **Still open (os-lint RED):** 3 stale bug reports 11–12 days old — Workouts-page delay (RE-OPENED at 07-06), 1RM 0.5kg shift, My Progress Strength tab live test.
- **Jake's 8 reports banked** — see the ledger; delete-template nav FIXED, the rest open/deferred/answered.

**2nd save (same day):**
- **Repo-root `CLAUDE.md` added** (coachapp `bed9625`) after reviewing a claude.ai starter-pack brainstorm — the repo had no in-repo grounding, so ritual-less sessions started blind. Guarded by a new os-lint `claude-md` drift check (claude-config `53103e8`); widening its regex caught + fixed a stale 8-module ref in `deploy-check`. Rest of the starter pack rejected (wrong stack / duplicates existing systems).
- **New to-do (deferred, Low):** pinch-to-zoom disabled — `index.html:5` `maximum-scale=1.0` blocks phone zoom (accessibility). Investigate the iOS-input-zoom trade-off + `font-size:16px` fix before changing.
- **Fixed the wiki `guide-coachapp-roadmap` Mermaid diagram** — two node labels had unquoted parentheses (`.limit(100)`, `(e600010)`); quoted them (same class as the 07-11 apostrophe break).

## 🐛 Session backlog — 2026-07-12 (session 26 cont. — 3 program bugs from real use + the empty-app beta blocker SOLVED)

_Same-day continuation. (A) Fixed 3 program-workflow bugs Jake hit in real use: stale view after assigning a program, "Update all same-named sessions" overwriting the whole workout, and program-workout edits not reaching the assigned calendar (now auto-sync — solo silent, clients confirm). (B) **Solved the empty-app beta blocker** end-to-end (brainstorm→spec→plan→implement→review): new-coach first-login seeds ~40 exercises + a sample workout + a sample program. Both workstreams' multi-agent reviews caught real issues, all fixed. 6 commits, all pushed, CI green, 133/2/0. Full detail in LOG._

- **✅ Done:** program-workflow fixes (app-programs v17, app-workouts v26); new-coach starter content (`js/starter-content.js`, app-core v4, +starter-content v1); `profiles.starter_seeded` migration (`scripts/add-starter-seeded-2026-07-12.sql`); spec + plan under `docs/superpowers/`.
- **🗓 Live-verify (Jake):** a real new-coach signup lands on a populated dashboard (only thing Playwright can't drive).
- **Decisions locked:** propagation matches by exercise NAME; program-edit auto-syncs assigned copies (solo silent / clients confirm); starter content is automatic + curated + app-side-with-a-flag; sample program not auto-assigned.

---

## 🐛 Session backlog — 2026-07-12 (session 26 — security & beta gates; a live storage breach caught by /deploy-check)

_Built the behavioural RLS audit and ran /deploy-check end-to-end for the first time. It found a **live cross-tenant storage leak** (any coach could read/delete any client's progress photos — bucket was private, but object policies were scoped by bucket_id alone). Fixed red→green. Also fixed Personal Bests (never displayed for anyone), caught 2 real runner bugs in a pre-push review, removed the progress-photos feature, wrote the ICO breach procedure. Full detail in LOG._

- **✅ Done:** behavioural RLS audit (`rls-audit.spec.js`, 4 probes + self-test); storage security (`storage-privacy.spec.js`); storage leak fix (`fix-storage-rls-2026-07-12.sql`); Personal Bests fix (app-progress v10); runner review fixes (app-runner v22); progress-photos feature removal (app-clients v5); ICO procedure; /deploy-check run + skill hardened (RLS + storage steps → behavioural).
- **🐛 New bug (open, Medium):** the suite erodes the E2E client's `workout_logs` (13→4 across runs) — a test deletes fixture logs it doesn't own without recreating them. Not breaking anything yet; Probe B's new lower bound will now fail loudly if it hits zero. Likely in runner.spec.js or client-workout.spec.js.
- **🚨 Confirmed open (CRITICAL):** the empty-new-coach beta blocker (see Beta prep) — still not started, top candidate for next session.
- **Coverage note:** Probe B is still vacuous on `performance_logs` (0 seeded rows); Probe C covers client_programs/1rms/check-ins but not performance_logs reads. Minor — seed those if extending.

---

## 🐛 Session backlog — 2026-07-11 (session 25 part 3 — runner table polish from real gym use, then planning)

_Jake used the runner in a real session and came back with 4 corrections. Committed as `8b9bb97` but **deliberately NOT pushed** at his request ("commit them but do not push, lets carry on with the scope") — it rides along with the next session's push after review._

| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | Target bar repeated its own column label — "RPE 8–9" *under* a column headed RPE | Low | **✅ Fixed (app-runner v21)** | Value now carries the number only. |
| 2 | **Remove the plate calculator** | Medium | **✅ Removed (app-runner v21)** | Shipped 2026-07-10 after "repeated requests" in competitor research; Jake used it in a real gym session and asked for it gone — noise, not help. Deleted outright (helpers + target-bar column + wizard hint + 5 tests). **The research said build it; the gym said remove it.** |
| 3 | `PREVIOUS` column values should sit under the column they represent | Medium | **✅ Fixed (app-runner v21)** | The separate 54px PREVIOUS cell squashed both numbers into one string ("140kg × 6"). It's gone — last session is now **ghost text** in the KG and REPS inputs, so each value sits directly under its own column, and KG/REPS get the freed width. |
| 4 | **KG/REPS must not auto-populate — ghost text only** | Medium | **✅ Fixed (app-runner v21)** | Reverses the v1 "1-tap repeat" pre-fill. A pre-filled value is indistinguishable from one you actually typed, so a set could be ticked off without the weight ever being confirmed. Rows now start empty. Two knock-ons handled: `toggleTableSet`'s "require reps" guard is now hit *routinely* rather than never (a silent no-op would read as a broken button mid-set → now warns), and `renderRunnerLastSession` used to **write** last session's values into `tableRows` when the async fetch resolved — a back door that would have re-introduced auto-fill. It now only repaints. |
| 5 | Barbell Back Squat rendering the wizard instead of the table | — | **Not a bug — Jake's own data** | He'd accidentally set the exercise unilateral; `_isPlainStrengthExercise` correctly excludes any unilateral set. He self-corrected ("may be a false flag from me"). No code change. |

---

## 🚨 Beta blocker surfaced 2026-07-11 — a new coach signs up to a COMPLETELY EMPTY app

**Status: 🗓 Needs scoping. Highest beta risk identified so far.**

Verified in code: `signUp` (app-core.js:332) creates the auth user, and the `handle_new_user` trigger creates **only** the `profiles` row (deliberately — the les-006 fix). There is **no starter data of any kind**. A brand-new PT lands on:

- **0 exercises** ← the hard blocker: you cannot build a workout without them, and the only route is typing each one in by hand
- 0 templates · 0 programs · 0 clients

**Jake has never experienced this.** His account has 200+ exercises accumulated over months of building. The app is excellent *once populated* and close to unusable *before* — and every beta PT starts at zero. A beta user's first session is data entry, not coaching.

**Scope (needs a decision):** ship a default exercise library on coach signup — how many, which ones, editable/deletable, and whether a sample template/programme comes with it (the "copy-paste simplicity + sensible default" product principle already in [[project-coachapp]] argues yes). Consider whether the existing `scripts/seed-test-data.js` exercise list is the starting point.

### Raised 2026-07-11, logged but NOT prioritised (Jake's explicit call — recorded so they aren't lost)

| Item | Why it may bite | Status |
|---|---|---|
| **Error monitoring / crash reporting** | `log.error` only reaches the *user's own* browser console. A beta PT hits a crash and Jake never finds out. The 2026-07-10 empty-phase crash was caught only because Jake personally hit it. Without this, the beta teaches you little about what's actually breaking. | Raised, deprioritised |
| **Backup / restore posture** | The 2026-07-11 data-loss bug destroyed real workouts. If that had hit a beta user, could they be restored? Supabase free-tier PITR is limited. Worth knowing the answer *before* strangers have data in there. | Raised, deprioritised |
| **Beta ops — feedback channel + invite email deliverability** | No in-app route for a beta PT to report a problem. And the invite Edge Function has only ever mailed Jake's own addresses — spam-folder risk untested, and a client who never receives their invite is a dead beta account. | Raised, deprioritised |
| **`max_rows = 200` cap** | Set during DB security hardening. A coach with >200 clients/templates/exercises silently truncates — this already bit once (Workouts page needed an explicit `.limit(100)`). | Known, unscoped |

---

## 🐛 Session backlog — 2026-07-11 (session 25 part 2 — the Library bridge + a CRITICAL data-loss bug)

_Jake's three asks turned out to be one problem: he'd built his personal program entirely with "+ Create new workout (this day only)", and there was no bridge from "built it in a program" to "reuse it". Building that bridge, the multi-agent review found a data-loss bug that was already live. Full repro steps in LOG._

| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | **🔴 DATA LOSS — Duplicate week → Generate weeks DESTROYED the Week-1 workout** | **🔴 Critical** | **✅ Fixed, proved red/green** | `_cleanupPhaseWeeksBeyond` deleted every template a stale week referenced with **no ownership and no still-referenced check** — while its sibling `deletePhaseWeek` had both (added 2026-07-10). A duplicated week shares the source's `template_id` (cheap-until-forked), so cleanup harvested Week 1's own template and deleted it. Also destroyed any **standalone library template** assigned into Week 2+, removing it from every program using it. Both call sites now share `_deleteOwnedUnreferencedTemplates`. Found by the review, not by any test. **Full reproduction steps in LOG.** |
| 2 | Copy program workouts → Library (per-workout + bulk, idempotent) | High | **✅ Shipped** | The missing bridge. app-workouts v25. |
| 3 | Tap-row workout picker replacing the native `<select>` | High | **✅ Shipped** | Answers Jake's "how do I tell three Upper Bodys apart" — an `<option>` can only hold plain text. Rows now show name + description + exercises. **Also closes the two long-open `<select>` complaints** (no feedback until opened; list grows unmanageable — both open since 2026-07-03). app-programs v16. |
| 4 | Duplicate week auto-extends the phase | Medium | **✅ Shipped** | Was hidden entirely on a 1-week phase with no explanation. `duration_weeks` bumped **after** a successful insert. |
| 5 | Review findings: name-dedupe dropped distinct workouts; idempotency guard failed open (`.maybeSingle()` errors on >1); `.ilike()` treated the name as a LIKE pattern; no picker re-entrancy guard; stale picker pool | Medium | **✅ All fixed pre-push** | |
| 6 | **BUG — deleting a program orphans its periodization week-clones into the reusable template pool** | 🟠 Medium | **Found, NOT fixed — needs a decision** | The phase cascade sets `generated_from_phase_id = NULL`, so a "Bench Press — W2" clone loses the only column marking it a derivative and becomes indistinguishable from a genuine standalone template. It escapes the ownership checks AND clutters the picker. **Same mechanism behind the picker clutter Jake reported 2026-07-10** (that fix removed program-owned templates from the pool, but not these orphans). Options: FK → CASCADE, or have `deleteProgram` sweep clones before deleting phases. |
| 7 | `scripts/seed-test-data.js` had never worked | Low | **✅ Fixed** | Omitted `exercise_name`/`exercise_type`, stringified `sets_json` (column is jsonb), and swallowed its own insert error while printing "Template exercises added". Now error-checked and idempotent. |

---

## 🐛 Session backlog — 2026-07-11 (session 25 — Personal Library page + a real cross-client RLS leak)

_Jake asked a question rather than reporting a bug ("personal account does not have workouts > templates & exercise library page... correct?") — he was right. Building it required an `is_personal` split on `workout_templates`, and auditing that split surfaced a genuine cross-client RLS leak nobody knew about. Full detail in LOG._

| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | **SECURITY — any client could read any OTHER client's workout template clones** | **🔴 Critical** | **✅ Fixed + confirmed red/green** | `Client reads workout templates` RLS policy scoped by `coach_id` alone, no `client_id` restriction. Clones carry `coach_id = coach, client_id = client`, so every client of the same coach matched. Reproduced live as a real client before fixing. Policy now requires `client_id is null`; own clones still resolve via `client_read_own_templates`. Also fixed a scalar `=` subquery that errors for any user with >1 clients row (the master account has two). |
| 2 | **Personal/solo had no route to the template builder at all** | **🔴 High** | **✅ Shipped** | `renderWorkouts` diverted solo into the read-only session view, so reusable workouts could only be made inline from a phase slot (locked to one day). New solo-only `library` nav entry → `renderWorkoutLibrary` (extracted, coach's page untouched). app-core v3 / app-workouts v24. |
| 3 | `workout_templates.is_personal` split | High | **✅ Shipped** | Required before the Library could open to solo — PT and solo share one `coach_id`, so a personal template would otherwise bleed into the real client-facing library. Mirrors `exercises.is_personal` (2026-07-10). 1537 existing rows → `false` (fix forward). **Deliberately NOT enforced at RLS** — see STATUS continuity block; enforcing it would break the client dashboard/calendar master-template embed. |
| 4 | **BUG — program day-slot picker showed the PT's whole template pool to solo** | High | **✅ Fixed same session** | Pre-existing, independent of the Library feature. `openProgram`'s query had no role split. |
| 5 | Review findings: `is_personal` clone carry-over was a silent no-op; 2 new tests could orphan `[E2E]` rows | Medium | **✅ Both fixed pre-push** | Clone sources were fetched via embedded selects omitting `is_personal` → `undefined` → dropped from insert → DB default. Not a live leak but a landmine. |

---

## How to read this

Each feature has a **status tag**: `✅ Done` / `🔧 In progress` / `🗓 Planned` / `💡 Future consideration` / `🐛 Bug`
Features inside a section are in priority order. Update status tags during each `/save`.

---

## 🐛 Session backlog — 2026-07-10 (session 24 — RLS audit, autosave + 3 more features, live picker bug, 3-agent review)

_A "quick RLS audit" scoped for 2 known gaps found a 3rd, more severe one nobody had flagged (`workout_template_exercises` had zero client-read policy at all). Built the 3 remaining scoped/requested features from the backlog (autosave, Delete week, plate calculator) plus %1RM rounding. Jake found a live bug mid-session (program picker showing every workout ever created, duplicates indistinguishable) — root-caused and fixed same session, not banked for later. Multi-agent review before push caught 3 more real issues. Full detail in LOG._

| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | **BUG — `workout_template_exercises` had no client-read RLS policy at all** | **🔴 Critical** | **✅ Fixed, confirmed end-to-end** | Same blind spot as the session-23 `client_programs` bug — solo's shared `auth.uid()` masked it. Broke `openSessionDetail` and the client Workouts-page accordion for every real (non-solo) client. Found by digging one level past what the audit was scoped to check. |
| 2 | **BUG — `client_1rms` had no write policy for a real coached client** | **🔴 High** | **✅ Fixed, confirmed end-to-end** | Only solo had INSERT/UPDATE/DELETE policies. Confirmed the 2026-07-03 to-do. |
| 3 | **BUG — program picker showed every workout ever created for a program, indistinguishable duplicates** | **🔴 High** | **✅ Fixed same session** | Found live via Jake's screenshot. `openProgram`'s template query had no `.is('program_id', null)` filter. Fixed + relabeled the inline "Create new workout" option "(this day only)" per Jake's explicit follow-up. |
| 4 | Exercises table PT/Personal scoping (`is_personal` column) | High | **✅ Fixed same session** | Found live: Personal-created exercises were bleeding into the PT-facing library. Existing data left as-is (Jake's call), only new creates separated going forward. |
| 5 | Runner session autosave, %1RM rounding, Delete week, plate calculator | — | **✅ All shipped** | See individual entries below in this file for full detail. |

---

_Picked up the in-progress hero card + Recent-sessions rename build from session 22's interruption. Fixing the resulting Playwright failures surfaced three real issues well beyond the original scope, each root-caused rather than patched around._

| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | **BUG — `client_programs` (+ 3 related tables) had no client-read RLS SELECT policy** | **🔴 Critical** | **✅ Fixed 2026-07-10 — confirmed working end-to-end** | Confirmed directly: a genuine (non-solo) client account read back **zero** rows from `client_programs`, even completely unfiltered. Jake applied the fix, but the first verification only checked the outer table — the app's real queries embed `programs(program_phases(program_phase_workouts(...)))` *inside* `client_programs`, and those 3 tables also had no client-read policy, so PostgREST silently nulled the embed and the dashboard still crashed even after the "fix" looked complete. Jake applied 3 more policies (`programs`, `program_phases`, `program_phase_workouts`); all 4 now confirmed live end-to-end via a fresh fixture test, not just a table-level check. Both previously-`test.fixme()`'d Playwright tests (hero card, this item) now pass for real; a new dedicated embed-chain regression test was added to catch this exact class of miss going forward. New standing skill `missed-check-to-test` created from this incident. |
| 2 | **PERF — Workouts page was always fetching a heavy, unused query** | **✅ Fixed this session** | `renderClientWorkoutsPage` fetched the flat templates list (up to 100 rows, nested exercise join) via `Promise.all` on every load, even when a program was assigned and the result was never used (only read in the no-program fallback branch). Restructured to fetch it only when `!hasProgram`. Worst-case impact was the personal/solo account specifically, since it shares the PT account's large historical orphan-template backlog. **This is very likely the same root cause as the still-open 2026-07-06 report** ("app runs slow... moving from dashboard to workouts page," STATUS.md open to-dos) that was never investigated — same page, same account type most likely to trigger it. Needs Jake's live re-confirmation that the slowness is gone. |
| 3 | **BUG — `deleteProgram()` was deleting shared templates it didn't own** | **✅ Fixed this session** | Root-caused via Playwright flakiness, not by inspection — a periodization test's throwaway program linked the shared "Push Day A" template into a slot, then deleted the program; `deleteProgram()` deleted *any* template referenced by the program's phases with no ownership check, destroying the shared template as a side effect. This is a real, pre-existing product bug: any coach who reuses a standalone template across programs and later deletes one of them would silently lose it. Fixed to only delete templates actually owned by the program (`program_id` match, or a periodization-generated week-clone from one of its own phases via `generated_from_phase_id`) — verified live that a periodization-generate-then-delete cycle now leaves zero orphaned templates. 3-agent review caught a real regression in the first version of this fix (it missed the periodization-clone case, which uses `program_id: null` too) — fixed before push. |
| 4 | Minor debris — ~13 leftover `[E2E] Periodization Test` throwaway programs + a handful of orphaned `Push Day A — W2/W3` template clones accumulated on the E2E test account across today's repeated test runs (before fix #3 above) | 🟢 Low, optional | Found, not cleaned up | A bulk-delete cleanup attempt was blocked by the harness's own safety classifier (pattern-matched delete against shared data I didn't create-and-track this session) — correctly so, since I hadn't asked first. Harmless (test account only), just cosmetic debris. Ask if you'd like it swept next session. |
| 5 | **BUG — live crash: a phase with zero sessions broke the Personal/solo Workouts page** | **🔴 Critical** | **✅ Fixed same-day (b79c152)** | Jake hit this live right after the hero-card build shipped. Root cause: pre-existing accordion code (`renderDays` in `app-workouts.js`) assumed at least one week/session always existed; a phase with zero `program_phase_workouts` (a normal state — a phase not yet fully built) made `weekNums` empty, crashing on `undefined.forEach`. Not part of the day's diff, so the diff-scoped multi-agent review never saw it; no test fixture (old or new) ever modeled a zero-session phase. Fixed with an explicit empty-phase message; new Playwright regression test added. Same-session meta-lesson as item 1 — banked into the new `missed-check-to-test` skill. |

---

## 🐛 Session backlog — 2026-07-08 (mid-session live-test report, session 22 3rd follow-up)

_Jake reported 6 more items while the Workouts-polish build (hero card + session-history rename) was in progress. Priority order below reflects severity, not report order._

| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | **BUG (data leak)** — PT-facing Workouts page showed workouts linked to the personal/solo account | **🔴 Critical** | **✅ Fixed, pushed this session** | Root cause confirmed: `renderWorkoutTemplates` (app-workouts.js) and `renderClientWorkoutsPage`'s flat-list fallback both filtered `.is('client_id', null).is('program_id', null)` but not `.is('generated_from_phase_id', null)` — periodization-generated week clones (e.g. "Bench Press — W2") have client_id/program_id both null too, so they leaked into the flat Templates list. Since solo shares `coach_id` with the PT account, Jake's own solo-program week-clones were cluttering his professional templates list. Not a cross-account RLS leak (coach_id scoping was intact throughout) — a query-completeness bug. Fixed by adding the missing filter, matching the pattern already used correctly elsewhere (`app-programs.js:589`, the phase day-slot assign picker). New Playwright regression test added. **Related, not fixed:** `startWorkoutRunner`'s freeform template list (app-workouts.js, when no specific templateId given) has no client_id/program_id/generated_from_phase_id filtering at all — same bug class, lower urgency since it's a template-picker list not a primary nav page; needs its own check. |
| 2 | **BUG** — Personal > Workouts page not loading | **🔴 High** | **✅ Fixed — was self-inflicted** | Caused by an in-progress hero-card edit (this session's own Workouts-polish build) left mid-flight by a session restart — `app-workouts.js` was calling two not-yet-defined functions. Completed the edit; confirmed via full Playwright suite. |
| 3 | **BUG** — "Log weight" button does nothing | **🟠 High** | **✅ Fixed, pushed this session** | Same shape as the "Log PB" bug fixed earlier this session: `showClientWeightForm()` toggled a DOM node (`client-weight-form`) that existed on the Dashboard pages, never on the Progress page's Body Weight tab it's actually clicked from. Added the form to `renderProgressWeight`; fixed `saveClientWeight`'s refresh target to detect Progress page vs. either dashboard (was unconditionally calling `renderClientDashboard`, which was also wrong for a solo user saving from their own dashboard's copy of this same form — same bug class fixed for `saveClientPB` earlier). |
| 4 | **BUG?** — Starting weight still doesn't populate in the weight logger | **🟠 High** | **Code confirmed correct — needs Jake's live re-check** | Re-verified the exact fix from earlier this session (`effectiveStarting = startingWeightKg ?? first.weight_kg`, `renderProgressWeight` app-progress.js) is intact and correct — no regression found. Three possible explanations for the continued report, in order of likelihood: (a) browser cache — testing before the `app-progress.js?v=7` deploy propagated; (b) testing on the PT-facing client-profile Weight tab (`renderClientWeight`), which has never had a "Starting" stat tile at all (only Current/Change/Entries) — only the client/solo self-view tab has one; (c) a genuinely different code path not yet found. Needs Jake to confirm which page he's testing on and do a hard refresh before this is investigated further — per the systematic-debugging rule, don't guess a second fix without new information. |
| 5 | Runner should round %1RM-calculated target weights down to the nearest 2.5kg | 🟡 Medium | **✅ Done 2026-07-10** | Turned out there's only one shared function (`_calcWeightFromPct`) behind every %1RM display site, so "round every displayed target" and "round the pre-fill" were the same code change, not two options as originally framed. Now floors to nearest 2.5kg. app-runner v18. |
| 6 | Plate calculator | 🟢 Low | **✅ Done 2026-07-10 → ❌ REMOVED 2026-07-11** | Shipped, then removed 8 days later at Jake's request after real gym use ("Remove plate calculator") — it was noise, not help. See Runner features table below. |

---

## 🐛 Session backlog — 2026-07-08 (Solo-account live-test pass, session 22)

_Jake sent a 12-item list of things noticed while using his own solo/personal account. Grouped by area, same convention as the 2026-07-05 backlog below. File:line refs were confirmed via 3 read-only Explore-agent passes over the actual code — "Confirmed" = root cause verified in code; "Needs live repro" = static reading couldn't confirm; "Needs scoping" = a real, currently-nonexistent feature needing a design conversation before building. **Same-day follow-up:** 3 confirmed bugs (#4, #7 bug half, #9) were fixed and pushed; the Cardio-blank report (#8) closed with no code needed; and every remaining "needs scoping" item except #11 was resolved via a dedicated scoping round (6 decisions via AskUserQuestion) — see each row's Status column._

### Area 1 — Programs
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | Program overview panel (collapsible: phase name, progression structure, sessions/week, phase notes) + edit name/description/duration | Medium | **Ready to build — scoped 2026-07-08** | **Decided:** duration is display-only (shown as the existing sum of phase durations) — no new edit capability, since name/description already work (`showEditProgramModal`/`saveProgram`, app-programs.js:722-761) and editing total duration directly would require auto-resizing phases with no clear rule for which one absorbs the change. Build: extend the existing periodization badge (app-programs.js:644-650) into a full per-phase summary; new `program_phases.notes` column (text, nullable — none exists today); derive sessions/week from `program_phase_workouts`. |
| 2 | Linear periodization: let the user enter a bespoke % per week (set phase duration, then one %-field per week) instead of the current Start%→End% formula | Medium | **Ready to build — scoped 2026-07-08** | **Decided:** new `periodization_type = 'custom'` value alongside `linear`/`undulating` — existing Linear (`{startPct,endPct}`, app-programs.js:979-988) stays untouched, no migration needed. New config shape `{ weekPcts: [...] }`, one entry per week. `_computePeriodizedPct` (app-programs.js:1115-1122) gets a new `'custom'` branch indexing `weekPcts[week-1]`. Configure modal renders N `%` inputs when `'custom'` is selected, N driven by `duration_weeks`. |
| 3 | "Delete week" button on a phase | Medium | **✅ Done 2026-07-10** | Built as scoped: renumbers weeks after the deleted one down by 1, decrements `duration_weeks`, same 4-step cleanup pattern as `_cleanupPhaseWeeksBeyond` filtered to one week. Multi-agent review caught a real gap before push — the ownership check alone wasn't enough because `duplicatePhaseWeek` shares `template_id` across weeks until forked-on-edit, so deleting one week could destroy a template a sibling week's surviving row still needed. Fixed with an extra "is this template still referenced by any surviving row" check; dedicated regression test added. app-programs v14. |

### Area 2 — Workouts
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 4 | **BUG** — editing a workout from the flat Workouts list (not via a Program phase) and saving throws an error | **High** | **✅ Defensive fix pushed 2026-07-08 (6d8c6a8)** — root-cause confirmation still open | `saveEditTemplate`/`deleteTemplate` (app-workouts.js) hardcoded `.eq('coach_id', currentUser.id)`; new `_resolveTemplateOwnerCoachId()` helper makes this role-aware (matching `startWorkoutRunner`'s established pattern), same as the read side already did. **Honest caveat:** on deeper trace, both reachable roles (solo, coach) already resolved to a matching coach_id before this fix — the original "asymmetric filter" hypothesis didn't fully hold up. The fix is a real hardening, not confirmed to be THE exact bug Jake hit. If it recurs, need his exact repro (program-assigned session vs. flat standalone list; the exact error text). |
| 5 | Hero card on the Workouts page — program name, phase/week, "next up", Start button — to cut clicks to start a workout | Medium | **✅ Shipped this session** | Built as `_buildWorkoutsHero`/`_renderWorkoutsHeroHtml` (app-workouts.js) — standalone functions, not shared with the dashboards' own inline hero logic (deliberate: avoids touching two already-working renders for a pure dedup benefit). Extended one step further than the dashboards' own hero: resolves the actual next scheduled session's `templateId` (first `program_phase_workouts` row in the current phase/week) so the Start button launches it directly instead of just linking back to this page. |
| 6 | Rename "Session history" → "Recent sessions", cap to last 5, date only | Low | **✅ Shipped this session** | Applied to both sites: `renderClientWorkoutsPage` and the PT client-profile's `renderClientWorkouts` — label changed, `.slice(0, 5)` added, rows simplified to date-only (dropped workout name/exercise count per Jake's exact spec). |

### Area 3 — Progress & Performance
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 7 | **BUG + redesign** — Body Weight: entered a starting weight but the graph didn't reshape; tiles should read Start → Current → Difference; add a dynamic "past 7 days" grid + collapsible month-sorted historic entries | **High** | **✅ Bug half pushed 2026-07-08 (6d8c6a8)** — redesign half still open | Fixed: "Starting" tile now prefers `starting_weight_kg` over the earliest `weight_logs` row; tile order is now Start→Current→Change; Y-axis clamp now activates with *either* starting or goal weight set (blended with actual logged range), not requiring both. **Still open (not built):** the dynamic "past 7 days" grid and collapsible month-sorted historic entries — that redesign half needs its own build, no scoping blockers. |
| 8 | **BUG?** — Cardio section is blank | Medium | **Closed 2026-07-08 — no code change** | Jake confirmed he hasn't actually logged a cardio session yet, so there's nothing to reproduce against. `renderProgressCardio` (app-progress.js:1137-1185) is fully wired to real `distance_m`/`duration_seconds` data — the empty state is correct as-is. Re-open only if a real logged cardio session still shows blank. |
| 9 | **BUG** — "Log PB" button does nothing; should open a modal to log a strength or cardio PB (exercise, weight/reps or duration/distance/time) | **High** | **✅ Wiring fix pushed 2026-07-08 (6d8c6a8)** — full modal redesign folded into item 10 | Fixed: the button's form (previously only existing on Dashboard pages, `app-dashboard.js:532,:800`) is now also rendered on the Progress page itself (`renderProgressPBs`, app-progress.js:1198+); `saveClientPB` (app-clients.js) now refreshes whichever view is actually showing it — also fixed a real solo-mode bug where saving from the solo dashboard's own PB form used to call the wrong (client) dashboard render. The bigger "proper strength/cardio type-select modal" ask is scoped into item 10's Personal Bests restructure below, not built standalone. |
| 10 | Fold Cardio + 1RMs into Personal Bests; restructure "Performance" into a "Per session" tab (recent-vs-previous comparison, expand to a progression graph) and a "Per exercise" tab (alphabetical, collapsible, live-search like the Exercise Picker); move the Workouts-page 1RM grid into this area too | Medium/Large | **✅ v1 shipped 2026-07-08 (e600010)** — supersedes the "Personal Bests / Performance merge" item below | Personal Bests now hosts the manual PB list, 1RMs (`renderClient1RMs`), and Cardio bests (`renderProgressCardio`) as sub-sections. Performance's old "Progressions" tab replaced with "Per exercise" (alphabetical, live-search reusing the exercise-picker filter pattern) and "Per session" (net new — lists `workout_logs` most-recent-first, expand a session to compare each exercise vs. its own previous occurrence, expand further for a progression chart). Moved the Workouts-page 1RM grid here, removing its now-dead backing query. Ships to client/solo self-view only — PT-facing `renderClientPerformance` (app-progress.js:304) is a fast-follow, not yet built. A 3-agent review caught and fixed 2 real issues before push: a stale-cache race where switching Client/Personal view mid-fetch could show the wrong client's data (fixed with the same token-guard pattern as `_oneRMRefreshToken`), and the new search box leaking a fresh set of Chart.js instances on every keystroke (fixed with tracked-and-destroyed chart instances). 6 new Playwright tests, 78/78 + pre-push 39/39 green. |

### Area 4 — Cross-cutting
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 11 | Client view and solo/personal account view should be almost identical UI/UX; personal account should have zero links to any PT or client page | Medium | **Deliberately deferred to its own dedicated session — decided 2026-07-08** | Re-checked today: the solo nav (app-core.js:211-218) and the PT/Personal view-switcher gating (app-core.js:129, :244-269) already have no leaked PT/client links reachable from a true solo account — that half already holds. The real remaining gaps are unchanged from the last check: `renderSoloDashboard` (app-dashboard.js:593-836) still lacks the sudo-banner and coach-branding-banner `renderClientDashboard` has (app-dashboard.js:339-352), and keeps its own unique 4-tile stats row instead of matching the client dashboard's layout. Jake explicitly chose not to resolve this in the same round as the other items — it ripples into calendar parity + redundant 1RMs and warrants focused attention on its own, not a quick call alongside everything else. |

**Status as of 2026-07-08, end of session:** #4, #7 (bug half), #9 pushed live (6d8c6a8). #8 closed, no code needed. #10 (Performance restructure) shipped (e600010). #5/#6 (Workouts hero card + session-rename) shipped this session. #1, #2, #3 are fully scoped and ready to build — no more open questions. Only #11 remains deliberately deferred to its own dedicated session. Remaining suggested build order: #1/#2/#3 (Programs trio, can be done independently of each other) → #7's redesign half (Body Weight 7-day grid + monthly collapse). See the newer "mid-session live-test report" backlog above for 6 additional items reported this session — 3 already fixed alongside this work, 2 need Jake's live input, 1 (rounding) is a new unscoped feature request.

---

## 🐛 Session backlog — 2026-07-05 (Jake's live-test pass)

_Jake live-tested a real gym session plus the wider app end-to-end and reported 16 items in one pass. Grouped by area so we can work through one at a time — priority is within each area, not across all 16. File:line refs were confirmed via a read-only code pass: "Confirmed" = root cause verified in the code; "Needs live repro" = static reading couldn't confirm it, next session should reproduce it live first._

### Area 1 — Runner (highest priority: live-blocking + silent-wrong-data bugs)
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 1 | "Save workout" produced an error at the end of a gym session | High | ✅ Built 2026-07-05 | Root cause was (a): `dbq`'s client-lookup fired a false-positive "Save failed" toast even though a safe fallback let the save continue — fixed with `{showUserError:false}` in 3 places (`saveRunnerSession`, `saveWorkoutSession`, `showLogSessionModal`). (b) was also real — fixed 2026-07-06 with a proper rollback (delete the partial `workout_logs`/`workout_log_exercises`/`workout_log_sets` rows on failure) after a review agent found the original fix let a retry silently duplicate the session instead of just re-enabling the button. |
| 2 | Swap/Add exercise + new rest time doesn't overwrite the original | High | ✅ Built 2026-07-05 | `_confirmRunnerExerciseFromModal` now derives `restSecs` from the entered rest field for both swap and add modes. |
| 3 | Trap Bar Jump UI inconsistent with sibling exercises in the same workout | Medium | ✅ Built 2026-07-05 | Live evidence with Jake overturned the original "broad jump" regex hypothesis — actual cause was `_isPlainStrengthExercise` deliberately excluding any exercise with `intensityMin` (%1RM) set, regardless of name. Fixed by removing that exclusion (the table's target bar already computed %1RM→kg correctly). Timed/unilateral exercises still excluded by design — no change there. |
| 4 | Runner delete-set button too close to the complete-set (✓) button | Medium | ✅ Built 2026-07-05 | `margin-left:8px` added between the two buttons. |
| 5 | Runner RPE field — header already says RPE/RIR, field itself redundantly repeats "RPE" | Low | ✅ Built 2026-07-05 | Mobile placeholder now shows a numeric range hint (`1–10`/`0–5`) instead of repeating "RPE". |
| 6 | Add Superset — replace the AMRAP button in the add-exercise modal with "Add Superset"; runner should treat the pair as one on-screen unit, tracking which set/exercise within the pair | Future | Scoping needed | Confirmed superset today is an exercise-level text field (app-workouts.js:906, ~1035), not a per-set pairing — a data-model change, not a label swap. Jake explicitly wants to scope this together. |

### Area 2 — Progress & Stats
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 7 | Workout-preview slider shows blank fields for cardio exercises | High | ✅ Built 2026-07-05 | `openSessionDetail`'s set-line builder now branches `isCardio` first and reuses the exact cardio-formatting logic already used by the template-card preview. |
| 8 | Entering a new 1RM with the same kg value as an existing entry silently changes the new value (e.g. entering 200kg when another exercise is already 200kg saves as 199.5kg) | High | Needs live repro | No rounding/dedup/nudge logic found anywhere in `save1RM`/`saveBig5OneRMs`/Epley paths (app-progress.js:229-290, 79-92). Likely DB-side (trigger/constraint) or a data-entry artifact. |
| 9 | Progress > 1RM "Update" — fields populate outside the modal, not inside it | Medium | ✅ Built 2026-07-05 | Root cause: `.modal-box` (used by 5 modal sites across app-progress.js/app-runner.js) had zero CSS definition anywhere — swapped all 5 to the correctly-styled `.modal` class. |
| 10 | Personal Bests and Performance tabs are redundant — move 1RMs into Personal Bests; repurpose Performance to track saved-workout-session progress over time (date completed + progress) | Medium/Future | Design decision | Jake: flesh out in a future session. Confirmed real overlap today between `renderProgressPBs` and `renderClientPerformance` (both surface `performance_logs`, app-progress.js:1109, 300). |
| 11 | Bodyweight graph Y-axis too fractional — should step in 0.5kg increments; lowest tick = goal weight, highest tick = 1kg above starting weight | Low (quick win) | ✅ Built 2026-07-05, fixed 2026-07-06 | 0.5kg stepSize added to Chart.js config. Goal/starting weight come from 2 new nullable `clients` columns (`starting_weight_kg`, `goal_weight_kg`). **2026-07-06 correction:** the 2026-07-05 build only wired this into the PT-facing `renderClientWeight` (client-profile Weight tab) — a review agent caught that the client/solo's own "My Progress → Body Weight" page uses a separate, untouched function (`renderProgressWeight`), so neither the goals form nor the Y-axis fix was reachable by the actual target user. Ported into `renderProgressWeight` too, and fixed a second bug found in the same review: the min/max calc assumed goal < starting (weight-loss only) — a weight-gain goal inverted the axis. Now uses `Math.min`/`Math.max` of the two values. |

### Area 3 — Personal / Solo account
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 12 | Personal account calendar — text overflows outside the calendar cells | Medium | Confirmed | app-calendar-goals.js:119-153 — zero calendar CSS classes exist anywhere (all inline styles); grid cells lack `min-width:0`, text is `white-space:nowrap` — classic CSS Grid overflow. |
| 13 | Personal > Workouts page shows 1RMs — redundant with Progress section | Low | **✅ Done 2026-07-08 (e600010)** — _was stale, ticked 2026-07-11_ | Already removed as part of item 10's Performance/Personal Bests restructure, which moved the Workouts-page 1RM grid into Personal Bests "removing its now-dead backing query" (LOG, session 22). Verified 2026-07-11: `client_1rms` has **zero** references in `app-workouts.js`. This row sat marked open for 3 days and was nearly rebuilt from scratch during planning — the reason `/save` now has a mandatory roadmap-reconciliation step (Step 3b). |
| 14 | Personal > Calendar should also show the workout-preview slider (parity with elsewhere) | Medium | Confirmed gap | `showClientDayDetail` (app-calendar-goals.js:215-263) never calls `openSessionDetail` — only renders exercise name + set count. |
| 15 | Personal account should structurally mirror the Client view exactly — the only difference should be no client/PT linkage | Future | Design decision | Jake's own architecture statement. Confirmed real diffs exist today: `renderSoloDashboard` (app-dashboard.js:583-819) lacks the sudo/branding banners `renderClientDashboard` (219-580) has, and has a unique 4-tile solo-stats row the client dash doesn't. Needs a dedicated session — affects items 13/14 too. |

### Area 4 — Dashboard
| # | Item | Priority | Status | Notes |
|---|---|---|---|---|
| 16 | Dashboard should show a program-name header (mirroring "Up next") with a "View program" button next to it | Medium | ✅ Built 2026-07-05 | Added above the "Up next" hero card on both client and solo dashboards, routes to the Workouts page. |

**Suggested order:** Area 1 (Runner) first — includes a live save-blocking error and a silent wrong-data bug — then Area 2 (Progress, its own data-integrity concern), then Area 3 (Personal/Solo), then Area 4 (Dashboard, quick win, slots in anytime).

**Two items need Jake in the room before building, not just code:** #6 (Add Superset — data model design) and #15 (Personal-mirrors-Client — architecture decision, ripples into #13/#14).

---

## PT-facing features

### Core shell
| Feature | Status | Notes |
|---|---|---|
| Auth — login / signup / session persistence | ✅ Done | |
| Coach dashboard shell | ✅ Done | |
| Sidebar + bottom nav | ✅ Done | Dashboard, Clients, Workouts, Calendar |
| Sign out | ✅ Done | |

### Client management
| Feature | Status | Notes |
|---|---|---|
| Client list | ✅ Done | |
| Add client | ✅ Done | |
| Client profile (tabs) | ✅ Done | Overview / Goals / Workouts / Weight / Performance / Programs / 1RMs. **Photos tab removed 2026-07-12** (Jake, "for now"); bucket + data retained, restorable from git history at app-progress v9. Removed alongside fixing a live cross-tenant leak in its storage policies — see CRITICAL.md storage section + `breach-procedure.md` §6. |
| Edit client details | ✅ Done | |
| Update client email (modal) | ✅ Done | |
| Invite client via email | ✅ Done | Edge Function — stamps user_id + invited_at at send time |
| Resend invite | ✅ Done | |
| Client compliance tracking | ✅ Done | Sessions per client this week — colour-coded card on PT dashboard |
| In-app PT→client messaging / notes thread | 💡 Future | Supabase Realtime; simple thread per client profile |
| **Client cannot self-detach from their PT except via a proper cancellation workflow** | 🗓 Planned, not scoped | 2026-07-05: surfaced while adding a client-role UPDATE policy on `clients` (for self-service weight goals). RLS in this app is row-level, not column-level, so any client-writable-row policy technically permits a client to update `coach_id` on their own record directly via the API, detaching themselves from their PT with no workflow, no notice, no PT-side visibility. Accepted as consistent with the existing trust model for now (same class of risk already present on `weight_logs`/`client_1rms`), but Jake flagged it as a real gap needing an actual designed cancellation/detach flow — not scoped yet. |

### PT dashboard
| Feature | Status | Notes |
|---|---|---|
| Stats row (active clients, sessions this week) | ✅ Done | |
| Recent activity feed (weight + workout logs, last 7 days) | ✅ Done | |
| This week's sessions compliance card | ✅ Done | |
| Goals due soon (next 14 days) | ✅ Done | |

### Calendar
| Feature | Status | Notes |
|---|---|---|
| Calendar page (monthly grid) | ✅ Done | |
| Month navigation | ✅ Done | |
| Add event (modal — title, date, type, client, notes) | ✅ Done | |
| Delete event | ✅ Done | |
| Per-client calendar view | 💡 Future | Filter calendar to one client's events |

### Exercise library
| Feature | Status | Notes |
|---|---|---|
| Global exercise library per coach | ✅ Done | |
| Add / edit / delete exercises | ✅ Done | |
| Muscle groups tagging | ✅ Done | |

### Workout templates
| Feature | Status | Notes |
|---|---|---|
| Create / edit / delete templates | ✅ Done | |
| Add exercises to template | ✅ Done | |
| Set form — AMRAP / Uni / Timed, Reps, Weight, %1RM, Rest, RPE/RIR, Tempo, Notes | ✅ Done | |
| Cardio set form — Pace/500m, Pace/km, HR Zone, Rest, Stroke rate, Duration, Distance | ✅ Done | |
| Template card set preview | ✅ Done | |
| Section labels ([WARM-UP] / [MAIN SET] / [COOL-DOWN]) | ✅ Done | |
| Log workout against template | ✅ Done | |
| View workout log | ✅ Done | |
| Template propagation → syncs client plan copies | ✅ Done | Apply-to-all now updates client plan copies via program_phase_workout_id FK |
| Custom exercise demo videos (YouTube link) | 🗓 Planned | Per exercise in library; competitor standard (TrainHeroic 1,500+, PT Distinction 1,656) |

### Workout runner
| Feature | Status | Notes |
|---|---|---|
| Real-time gym logger | ✅ Done | |
| Cardio mode + interval timer | ✅ Done | |
| AMRAP / EMOM / circuit timer mode | 🗓 Planned | Dedicated timer for AMRAP (count-down), EMOM (minute boundary beep), circuit (rounds × work/rest); competitor standard on TrainHeroic |
| Rest timer | ✅ Done | |
| Bodyweight / assisted / superset support | ✅ Done | |
| Timed sets — Start button + countdown overlay | ✅ Done | v169 — ▶ Start → fullscreen ring timer → auto-fills duration on complete |
| Unilateral L/R logging | ✅ Done | |
| Set X of Y counter + progress dots | ✅ Done | Positioned below last-session strip, above inputs |
| Session summary / finish screen | ✅ Done | PR badge, stats row, exercise cards |
| Runner target chips (pace, duration, stroke rate, rest HR) | ✅ Done | |
| Client notes per exercise | ✅ Done | Saves to workout_log_exercises.client_notes |
| Voice cue at 10s rest + beeps at 3/2/1 | ✅ Done | v169 — Web Speech API; unlocked on first gesture |
| Last session data accuracy | ✅ Done | v169 — fixed stale query ordering |
| Stats bar removed — timer only in header | ✅ Done | v169 — cleaner runner layout |
| **Runner redesign → Hevy-style table logger** (supersedes "set input pre-fill") | ✅ Done (v1) — **revised 2026-07-11** | 2026-07-02, pushed 6e6402a: all-sets-visible table, tap-✓ to complete, non-blocking rest timer. **v1 = plain strength only**; cardio/timed/unilateral stay on the wizard — phase 2 still 🗓 Planned. **Two v1 design choices were REVERSED on 2026-07-11 (app-runner v21) after Jake used it in real gym sessions:** (a) the `PREVIOUS` column is gone — last session is now **ghost text** in the KG/REPS inputs, so each value sits under the column it belongs to instead of being squashed into one cramped 54px cell; (b) **pre-fill / "1-tap repeat" removed** — rows now start EMPTY. A pre-filled value is indistinguishable from one you actually entered, so you could tick a set off having never confirmed the weight was right. Logging accuracy beat tap-count. See STATUS "What's working" + "In progress" for the two known v1 gaps (superset auto-switch, bodyweight live-verify) |
| **Runner redesign phase 2** — extend table (or equivalent) to cardio/timed/unilateral/%1RM exercises | 🗓 Planned | Scope not yet defined; v1 deliberately excluded these to ship the strength-only case first |
| **Plate calculator (what to load on the bar)** | **❌ REMOVED 2026-07-11** (shipped 2026-07-10, app-runner v19; deleted app-runner v21) | Built after repeated requests (2026-07-02 competitor research): standard 20kg bar + greedy per-side breakdown, as a PLATES/SIDE column in the strength table's target bar plus a live hint under the wizard's weight input. **Jake used it in a real gym session and asked for it to be removed outright** — in practice it was noise, not help. Deleted rather than hidden behind a flag (`_calcPlateBreakdown`/`_updatePlateBreakdown`/`_PLATE_SIZES` and its 5 tests all gone). **Lesson worth keeping: "repeatedly requested in research" did not survive contact with real use** — the only test that mattered was Jake actually lifting with it. |
| **Improve workout-tracking visuals + underlying data model** | 🗓 Planned | Jake, 2026-07-04: wants a better look at how a workout is tracked in the runner, and to reconsider where/how that data is stored. The "where it's stored" half is now done (runner autosave, below); the visuals half is still unscoped — needs a sounding-board session. |
| **Runner session autosave (localStorage draft)** | **✅ Done 2026-07-10** | Built as scoped: localStorage-only draft, checkpoint on every `renderRunner()` + a 10s safety-net tick, key `_runnerDraft_<clientId>`, captures `loggedSets` + `tableRows`, same-day staleness cutoff, resume/discard confirm modal. **Refined from the original scoping note:** wired into `launchRunner()`, not `startWorkoutRunner()` — reading the real code showed `launchRunner` is the true single choke point both the fast templateId path and the setup modal's own Start button funnel through, so `startWorkoutRunner()` alone would have missed the modal path. Cleared inside `discardRunner()` (covers both abandon and post-save). Multi-agent review caught a real gap before push: resumed drafts skipped the audio/speech-unlock gesture, so a resumed session's rest-timer cues could silently never fire — fixed. Fixes the 2026-07-04 live incident (runner freeze + forced reload wiped an entire in-progress gym session). app-runner v17. |
| **Rest time not overwritten on swap/add** | 🐛 Bug (confirmed 2026-07-05) | Swap mode never reassigns `ex.restSecs`; add mode hardcodes 90s, ignoring the entered value. See session backlog Area 1 #2. |
| **App feels slow saving an updated workout, and navigating dashboard → Workouts page** | ✅ Done 2026-07-07 | Root cause of save: `saveRunnerSession`/`saveWorkoutSession` inserted one exercise + one sets-batch per exercise, sequentially (up to ~26 round trips for 6 exercises) — batched into 2 inserts total, measured 14 requests/4.7s → 4 requests/1.1s. Root cause of Workouts-page load: both template queries had no `.limit()`, riding the 200-row server cap — added `.limit(100)` to both; also cleaned up 103 confirmed-orphaned `workout_templates`. Pushed 444d0f3. |
| **Exercise picker modal shrinks/drifts toward the bottom of the screen as search results narrow** | ✅ Done 2026-07-07 | Jake reported live: as fewer results match a typed query, the modal (and the search input itself) visually shrinks and slides down the screen, crowding the on-screen keyboard on mobile. Root cause: `max-height:85vh` with no fixed `height`, combined with mobile's bottom-anchored overlay. Fixed with `height:70vh` so the box stays constant size/position regardless of result count. Pushed 682f86f. |
| **Add Superset (redefine current AMRAP button)** | 🗓 Planned, needs scoping | 2026-07-05: Jake wants the add-exercise modal's AMRAP button replaced with "Add Superset" — pairs two exercises so the runner shows both on one screen and tracks sets/exercise within the pair. Confirmed today's superset field is exercise-level text, not a per-set pairing — data-model change, not a label swap. Supersedes/extends the "Superset auto-advance" gap already tracked in [[coachapp-runner-architecture]]. See session backlog Area 1 #6. |

### Weight tracking
| Feature | Status | Notes |
|---|---|---|
| Log weight (kg) + body fat % | ✅ Done | |
| Stats row (current / starting / change) | ✅ Done | |
| Chart.js line chart | ✅ Done | |
| Log table | ✅ Done | |

### Performance / PB tracking
| Feature | Status | Notes |
|---|---|---|
| Log performance (4 categories: strength / cardio / body metric / benchmark) | ✅ Done | |
| Best per exercise grouped by category | ✅ Done | |
| PB badge (gold) | ✅ Done | |
| Expandable history per exercise | ✅ Done | |
| Chart.js progression chart per exercise | ✅ Done | |
| Delete log entry | ✅ Done | |
| **Personal Bests / Performance merge** | **✅ v1 shipped 2026-07-08 (e600010)** | 2026-07-05: Jake — the two tabs are redundant. Move 1RMs into Personal Bests; repurpose Performance to track saved-workout-session progress over time (date completed + trend). See roadmap session backlog Area 2 #10. **2026-07-08:** shipped as scoped — Personal Bests now hosts manual PBs + 1RMs + Cardio bests; Performance split into Per session (net new) / Per exercise (alphabetical + search). PT-facing client-profile tab (`renderClientPerformance`) still uses the old form — fast-follow, not yet done. See session backlog 2026-07-08 Area 3 #10 for full detail. |

### Goals
| Feature | Status | Notes |
|---|---|---|
| Create / edit goals with target dates | ✅ Done | |
| Milestones | ✅ Done | |
| Check-ins | ✅ Done | |
| Goals due soon on PT dashboard | ✅ Done | |
| Goals overhaul — granular mini-goals and milestones | 🗓 Planned | Medium priority |

### Programs (phase-based training plans)
| Feature | Status | Notes |
|---|---|---|
| Programs schema (4 tables) | ✅ Done | |
| Programs UI — create / edit / delete programs | ✅ Done | |
| Program phases (weeks, ordering) | ✅ Done | |
| Assign workout templates to phase days — inline 7-day grid, no modal | ✅ Done | 2026-07-01 — replaced the old "+ Assign workout" modal (day→session→template, one at a time) with an always-visible searchable grid on the phase card; picking a template assigns immediately. Built to cut repetition, not just polish the old picker — matches the new "efficiency is the platform's spec" standing principle. |
| Assign program to client with start date | ✅ Done | |
| Edit start date | ✅ Done | Shifts entire calendar |
| Remove program from client | ✅ Done | |
| PT client programs accordion (Phase → Day → SESSION N/M → exercises) | ✅ Done | |
| Client plan editing (PT edits client's sessions) | ✅ Done | Apply-to-all propagation + client copy sync |
| Auto-create calendar events from program schedule | 💡 Future | |
| Periodization (Linear / Undulating) — phase-level %1RM automation | ✅ Done | 2026-07-01 — generatePhasePeriodization(); propagates to already-assigned clients |
| 1RM system — inline runner prompt, Epley estimator, Big 5 quick-start, post-session suggestion | ✅ Done | Found already built and live 2026-07-01 (STATUS.md v181 entry was stale) — `showRunnerOneRMSheet`, `showAdd1RMModal`, `saveBig5OneRMs`, `showPostSessionOneRMModal` |
| 1RM assignment-time missing-1RM check | ✅ Done | 2026-07-01 — PT quick-fills known/estimated 1RMs inline on the Assign modal when a program needs lifts the client doesn't have; covers both assign entry points + solo; never blocks. Playwright smoke tests added. |
| Exercise identity — move from free-text name matching to exercise-library-linked (exercise_id) entry | ✅ Done 2026-07-06 | Jake reported the runner's "previous session" data goes missing when the same exercise is typed slightly differently across two workout templates. Confirmed root cause: exact-string-match fragility, not workout isolation. **Built:** a real `exercise_id` FK on `workout_log_exercises` and `client_1rms` (`workout_template_exercises` already had one), with a name-match fallback for older/unlinked rows — wired through `saveRunnerSession`, `saveWorkoutSession`, `save1RM`, `saveBig5OneRMs`, `_getProgramOneRMStatus`, `fetchRunnerLastSession`, `_lookupClientOneRM`. **Historical data migrated:** one-time SQL seeded the (previously empty) exercise library from real usage and linked 4777 template exercises, 27 logged exercises, 18/19 1RMs; Jake reviewed his actual exercise list and told me which spelling variants to merge (Close Grip Pulldown, RowErg, Trap Bar Jump, etc.). **New shared Exercise Picker** (search-as-you-type, explicit "Create new exercise", collapsible archived section) replaces the old dropdown+free-text entry everywhere — workout builder (add + edit), runner swap/add, 1RM entry. Archive/unarchive added to the Exercise Library management page. The old "1RM lifts quick-pick + auto-scroll" dropdown shortcut was dropped (Jake confirmed fine with this) — the underlying %1RM calculation itself was untouched and unaffected. 4 real bugs found and fixed via multi-agent review: a race condition wiping the search box, two missing RLS policies (clients had no INSERT/SELECT on `exercises` at all), an apostrophe-escaping bug breaking the picker for names like "Farmer's Carry", and two double-tap duplicate-row races. **Pushed** (`1526704`). First push attempt was blocked by the pre-push hook's smoke-test pass — initially misdiagnosed as environmental flakiness, but the real cause was a genuine test-suite race condition (`loginAsClient` not waiting for the client dashboard to finish rendering, unlike `loginAsPT`); fixed and pushed separately (`31698fe`). |
| Progression rules engine | 💡 Future | Auto-calculates target weight per session |
| Individual session skip/move on client calendar | 💡 Future | Defer until real PT usage data |

### Branding / UI
| Feature | Status | Notes |
|---|---|---|
| Branding — logo upload, display on dashboards | ✅ Done | v151 — private logos bucket, signed URLs, sidebar + PT/client dashboards |
| UI consistency pass | ✅ Done | SESSION N/M labels + exercise lists unified across all surfaces (v143) |
| Progress tabs — pill grid (no scroll) | ✅ Done | v171 — flex-wrap pills, all 5 visible at once on mobile |
| **Dashboard consistency pass (PT/client/solo)** | ✅ Done | 2026-07-05 (main.css v4, app-dashboard v2): `.dashboard-card`/`.card-header`/`.card-title` were used ~37× across all 3 dashboards with zero CSS definitions (rendered with no background/border/shadow) — added real rules. Consolidated 3 duplicated grid `<style>` blocks into one `.dashboard-split-grid`. Fixed 4 bare `class="btn"` Cancel buttons (no matching CSS) to `.btn-secondary`. Replaced hardcoded hex colors with design tokens. Fixed PT stat strip (no mobile override, cramped at 480px) and solo stat strip (`display:none` below 640px, vanished entirely) to pair up on mobile instead. |
| **App-wide undefined CSS vars/classes — found 2026-07-05** | ✅ Done 2026-07-23 (`c72eb14`) | `var(--surface-2)` ×48, `var(--bg-accent)` ×3, `var(--text-accent)` ×3 — 54 references, defined nowhere, so every one silently rendered transparent/inherited. Audited all 48 `--surface-2` sites first (every one a `background`), then defined all three in `:root`. Shipped alongside the builder repaint (15 hardcoded greys → 0). main.css v5→v6. 18 days open. **Awaiting Jake's eyes** — it changes surface styling app-wide, not just the builder. |
| Metric / imperial toggle | 🗓 Planned | Medium priority |

### Leaderboards
| Feature | Status | Notes |
|---|---|---|
| Client leaderboard (most weight lost, heaviest squat, etc.) | 💡 Future | |

---

## Client-facing features

### Client dashboard
| Feature | Status | Notes |
|---|---|---|
| Client login (invite acceptance) | ✅ Done | |
| Client dashboard (this-week banner, check-in, recent sessions, weight chart) | ✅ Done | |
| Client calendar | ✅ Done | Monthly grid; program workouts mapped to dates; day detail modal |
| Client Workouts page | ✅ Done | Phase → Day → SESSION N/M → exercise list → Start button |
| Client logs own weight | ✅ Done | |
| Client logs own workouts (runner) | ✅ Done | |
| Client views own goals + progress | ✅ Done | |
| Client adds own PBs | ✅ Done | |
| Client video submission (form review) | 💡 Future | |

### Solo user / Personal account (self-coached)

_Jake's master account gets a third "Personal" pill. Solo user's own client record has `coach_id = NULL` (not `auth.uid()` — corrected 2026-07-04, verified against `app-core.js:132`'s `.is('coach_id', null)` lookup), identified instead via `user_id = auth.uid()` + null coach_id. `window._soloClientId` holds that record's id. Solo write access to tables like `client_1rms`/`client_programs` needed its own explicit RLS policies (added 2026-07-01) precisely because they don't inherit coach-scoped policies through a `coach_id` match._

| Feature | Status | Notes |
|---|---|---|
| Third pill — Personal view in master account switcher | ✅ Done | `switchView('solo')` — both desktop sidebar + mobile |
| Solo nav + dashboard | ✅ Done | No Clients section; personal dashboard |
| Self-coached client record (coach_id = auth.uid()) | ✅ Done | Jake West record migrated; severed from PT account |
| One-time data migration SQL | ✅ Done | 8 tables cloned from old Jake West client record |
| Program self-assign flow | ✅ Done | Hyrox Hero assigned to solo account |
| Solo weight + PB tracking | ✅ Done | |
| Solo workout runner | ✅ Done | |
| My Progress — 5 tabs incl. 1RMs | ✅ Done | v170 — Body Weight, Strength, Cardio, Personal Bests, 1RMs |
| Delete old Jake West PT-account client record | 🗓 Planned | Clean up PT dashboard — only real/test clients should remain |
| Upgrade path: solo → PT-coached | 💡 Future | Solo accepts PT invite, coach_id stamped, retains history |
| Upgrade path: solo → becomes a PT | 💡 Future | Role change; gains coach dashboard + client management |
| **Personal account mirrors Client view exactly** | 🗓 Planned, needs scoping | 2026-07-05: Jake's own architecture statement — the only difference between Personal and Client should be no client/PT linkage. Confirmed real diffs exist today (`renderSoloDashboard` lacks the sudo/branding banners `renderClientDashboard` has; solo has a unique 4-tile stats row). Needs a dedicated session — ripples into the redundant-1RMs-on-Workouts-page and calendar-preview-parity items. See roadmap session backlog Area 3 #15. **2026-07-08 re-confirmed:** Jake restated this more strongly — solo should have zero links to any PT/client page. Re-checked: the solo nav and view-switcher gating already have no leaked links, so that half already holds. The gaps are unchanged (sudo/branding banners, unique stats row) — see session backlog 2026-07-08 Area 4 #11. |

---

## Business / platform features

| Feature | Status | Notes |
|---|---|---|
| Branding / custom logo | ✅ Done | v151 — coach_branding table, RLS, logo signed URLs, business name |
| Marketplace — sell programs to non-clients | 💡 Future | |
| Assistant coach support (multi-coach) | 💡 Future | |
| MFP / nutrition CSV import | 💡 Future | |

---

## Infrastructure / tech

| Item | Status | Notes |
|---|---|---|
| Supabase project (avilxuiacmtgeoxxhfhc) | ✅ Done | |
| RLS policies — coach-scoped | ✅ Done | |
| RLS policies — client-scoped | ✅ Done | Optimised 2026-06-23; (SELECT auth.uid()) caching |
| Audit log (DB-side triggers on 17 tables) | ✅ Done | |
| Structured console logging (JS-side) | ✅ Done | |
| Silent failure audit + dbq() wrapper | ✅ Done | |
| Edge Function — invite-client | ✅ Done | |
| Git (master branch) + GitHub Pages deploy | ✅ Done | Auto-deploys on push |
| PWA — manifest.json + service worker | 🗓 Planned | Makes app installable on iOS/Android from browser; no app store required; use PWABuilder (free, Microsoft) |
| Native app (Capacitor wrapper) | 💡 Future | Wraps existing JS in native shell; enables push notifications + app store listing |
| Pre-push git hook + GitHub Actions CI | ✅ Done | 10 static checks + Playwright (local only; CI skips — no credentials) |
| Playwright E2E suite | ✅ Done | 18 tests; 10 passing in CI, 8 solo skipped (need Jake's account); runner + client + auth + settings flows |
| SQL safety skill | ✅ Done | |
| Vault → GitHub backup | ✅ Done | jakendwest-ops/vault; auto-pushed on /save |

---

## Beta prep — target date: **31 July 2026**

_Date pushed from the original Jul 22–31 window to a single date, 31 July, per Jake's 2026-07-06 decision. Sessions 9–12 (Jul 2–3) went entirely into the runner redesign — a deliberate, research-backed pivot (approved after the Jul 2 competitor research), not drift. The pre-beta gates below still haven't moved and remain the real risk to hitting even the new date._

- ✅ **BETA BLOCKER SOLVED 2026-07-12 (`90c6d9e`):** a brand-new coach's first login now seeds ~40 exercises + a sample workout + a sample program (`js/starter-content.js`, gated by `profiles.starter_seeded`, resumable). Not auto-assigned; examples deletable. Live-verify pending: a real new signup landing on a populated dashboard.
- **⚠️ Pre-beta gates — action before 31 July:**
  - ✅ **ICO breach-notification procedure** — **Done 2026-07-12** (`breach-procedure.md`; CRITICAL.md now ✅).
  - ✅ **`/deploy-check` run end-to-end** — **Done 2026-07-12**, first time ever. 8/9 gates green; found + fixed a live cross-tenant storage leak in the process. One manual gate left: a live client smoke test (Jake-only).
  - ✅ **Supabase redirect URL** — confirmed present (`…github.io/coachapp/**` + Site URL). 2026-07-12.
  - **Delete old Jake West client record** from the PT account (see "Solo user" section)
  - **Supabase Pro upgrade** — unlocks leaked-password (HaveIBeenPwned) protection
- Full walkthrough, Playwright suite (127 passing), `/deploy-check` — ✅ redirect-URL + RLS + storage gates done
- **Beta invites: single date, 31 July** (previously staggered Jul 25/28/31 — simplified to one date)
- ✅ **1RM system built + tested** (done 2026-07-01) — drives %1RM in programs
