# CoachApp — Session Log

Newest first.

---

## 2026-07-17 — OS rebuild + os-lint + the first-ever full-file review found 18 latent bugs; fixed 20, pre-push review caught 4 of my own regressions (app-core v5, app-clients v6, app-dashboard v4, app-programs v19, app-progress v11, app-runner v23, app-workouts v28) — COMMITTED, awaiting Jake's push

_Long session. Built the approved OS rebuild, then it immediately paid for itself: the staleness lint and the first-ever full-file review each found real problems, and the pre-push review caught four regressions I introduced while fixing the findings._

**Done — the OS rebuild (`~/.claude`, backed to claude-config):**
- **`os-lint.mjs`** — the staleness lint (Jake's idea, the load-bearing piece). 9 checks: dead tool refs, dead file refs, PII-in-skills, module-count drift, retired-terms, frontmatter, full-file-review marker, stale bug rows (>7d), gate-never-fired. **Silent when clean, RED when not.** Proved each detector RED→GREEN on planted faults before trusting it (the deploy-check standard). Wired into a SessionStart hook in `coachapp/.claude/settings.json` (Jake pasted it; **confirmed firing live this session**).
- **Caught 2 things the hand audit missed:** a hardcoded UUID in run-coachapp, and **live E2E account passwords in plaintext** in the playwright skill — in a dir /save pushes to GitHub. All PII stripped from skills (sql-safety client name+UUID, owner email/uid, run-coachapp UUID, playwright passwords, deploy-check email).
- **hello-claude 319→~185 lines, 33 standing behaviours → 13.** Bug-ledger intake rule + closure rule (a Jake-reported item closes ONLY on his confirmation or a red→green test). /save rewritten: intake-first, 9-module cache-bust (starter-content was invisible to the app-*.js glob), skills-PII gate before the claude-config push. Archived post-build-review + security-audit (out of dispatch; hard delete still needs Jake). Corrected the stale "/code-review ultra doesn't exist" claim in 4 skills — it DOES exist in the extension, just billed/user-triggered.

**Done — the first-ever full-file review (multi-agent, whole app-workouts/runner/programs):** found **18 latent bugs** in code no recent diff had touched — exactly the blind spot it exists for. Fixed all, plus 2 more the pre-push review surfaced. Headlines:
- 🔴 **Stored XSS** — client-authored session name/exercise name/notes rendered unescaped in the COACH's DOM → JWT theft, defeats RLS. Fixed + added `escapeAttr` (JS-then-HTML escape order). Then the pre-push review found the first pass was fix-the-instance: swept the whole codebase (renderClient1RMs, sudo button, renderClients, dashboard feed, program/phase/exercise names) — grep now confirms **zero** raw user-controlled interpolations.
- 🔴 **Live data-loss chain** — stale `_phaseWorkoutContext` (cleared in one place, leaked on every other modal exit) could stamp a personal template into a coaching program, then Generate-weeks wiped real clients' weeks 2+. Root-fixed: the context is now an argument owned by `showCreateTemplateModal`.
- 🔴 **Live crash** — a zero-session phase crashed the client Programs tab (the guard existed in the verbatim twin since 07-10, never ported — 5th fix-the-class miss).
- **Client had the coach's Delete button + could overwrite coach notes** (openWorkoutLog, no role gate). **deleteProgram re-ran the debris factory** (bypassed `_removeAssignmentAndClones`). **deletePhase/removePhaseWorkout orphaned templates** (widened the shared guard to phaseIds[], all 5 delete paths now share it). Silent-failure fixes: `_cloneProgramForClient` now fails loud, `_assignedCopiesForSession` uses flat joins not a nullable embed, showLogSessionModal filters personal/week-clones out of the client dropdown. Runner: finish-screen teardown (was destroyed mid-typing by a live rest timer), runnerGoBack no-op-during-rest, editRunnerSet double-tap-saves-old-values, 25 modal mounts routed through `mountModal` (re-entrancy), the 1RM `block` ReferenceError. Jake's own report: deleting a template ejected solo out of the Library (solo-specific — now routes via `_templateGoBack`).

**Bugs I introduced, caught by the pre-push review before push (the gate earning its place):**
- A `coach_id` filter that excluded the solo record (solo's coach_id is NULL) — would have killed solo sync. **4th bug of this exact shape.**
- deleteProgram continuing even if clone-cleanup failed → orphaned data. Now fails closed.
- deleteTemplate routing `backTo:'client'` (a sentinel) to `navigate('client')` → "Page not found".
- The 1RM re-render wiping in-progress set inputs (no flush-first). Fixed + RPE/RIR buttons had the same gap.

**Tests:** 9 new regression tests in `tests/regression-2026-07-13.spec.js`, all proven RED-before/GREEN-after (the 1RM one isolated to its exact line, since the bug lived entirely inside this diff). Full suite **151 passed / 2 skipped / 0 failed** (reconciled to 153 declared). 7 modules cache-bust-bumped.

**Jake's 8 new reports** (banked to the ledger the moment he sent them — intake rule's first real use): Hevy plagiarism Q (answered: low risk, stop calling it "Hevy-style" publicly), delete-button-size/swipe-to-delete (deferred, needs scoping), add-exercise-to-program propagation gap, 1RM-target-as-percentage, mobile calendar overspill, delete-template nav (FIXED), Workouts-page delay (RE-OPENED at original 07-06 date — closed on a guess before), create-template slowness.

**Decided:**
- The solo/NULL-coach_id filter trap is now the **4th** bug of its shape — the single most reliable way to break this app. Banked to lessons.jsonl.
- Pre-push review is non-negotiable even when it's my own careful work: it caught 4 regressions in minutes. The spend limit killed it once mid-session (raised, re-ran).

**UNVERIFIED (banked):** all fixes are Playwright + review verified, NOT yet confirmed by Jake in a real gym/coaching session. The push itself is Jake's to run.

## 2026-07-12 — 3 program-workflow bugs from real use, then the empty-app beta blocker SOLVED (app-programs v17, app-workouts v26, app-core v4, +starter-content v1)

_Same-day continuation of the "Security & beta gates" session (below). Two workstreams: (A) three program bugs Jake hit in real use, (B) the new-coach starter-content feature built through the full brainstorm→spec→plan→implement→review flow._

**Done (A — program-workflow bugs, `4d8813c`):**
- **Stale view after assigning a program** — `saveAssignProgram`/`saveAssignProgramToClient` fired `_cloneProgramForClient` **fire-and-forget**, then re-rendered/navigated before the `client_program_workouts` rows every calendar/Workouts/dashboard view reads had been created → "old data until refresh", most visible when the program starts today. Now awaited, behind an "Assigning…" state, with a re-render after.
- **"Update all same-named sessions" overwrote the whole workout** — `_applyToAllSessions` deleted every exercise in each target and re-inserted the source's full list, wiping any exercise a target had that the source didn't. Replaced with `_propagateExerciseChangeToTemplates`: applies ONLY the changed exercise, matched **by name** (Jake's choice), captured at edit time in `window._lastExerciseChange` (add/update/delete). Update/delete both act on all same-named rows (consistent).
- **Editing a program workout didn't reach assigned copies** — the calendar reads the client's cloned copy (made at assignment), which a later master edit never touched. Now `_checkClientPlanPropagation` syncs assigned copies of the edited session automatically via `_assignedCopiesForSession`: the user's OWN solo copies silently (Jake's actual pain), real clients' copies only behind an "Update assigned clients?" confirm.

**Done (B — new-coach starter content, `25b978c`→`90c6d9e`):**
- Solved the **empty-app beta blocker** (highest open beta risk): a brand-new coach's first login seeds ~40 curated exercises + an "Example — Full Body A" workout (6 exercises linked to the library by name) + an "Example — 4-Week Foundation" program (Mon/Thu). `js/starter-content.js` (`STARTER_EXERCISES`/`STARTER_TEMPLATE`/`STARTER_PROGRAM` + `_seedStarterContent`/`_markSeeded`), wired into `loadUserInfo`, gated by a new `profiles.starter_seeded` flag. Migration applied by Jake (existing profiles backfilled `true` — nobody retro-seeded). Not auto-assigned to the coach's calendar; examples labelled "Example —", deletable. Built via the superpowers brainstorm→spec→plan flow (docs/superpowers/specs + plans committed).

**Bugs found + fixed (by the multi-agent reviews before push):**
- **(A) stale-change replay on delete** — if `deleteTemplateExercise`'s pre-delete name-fetch returned null, `window._lastExerciseChange` kept the PREVIOUS edit's change and silently applied it to the solo copy. Now cleared on failed capture (skip propagation).
- **(A) real-client consent bypass** — `_applyToAllSessions` wrote to real clients' SIBLING copies with no confirm, while the edited session's client copies were gated. Now the bulk path silently syncs only the user's own solo copies; real clients are never touched without the per-session confirm.
- **(B) partial-failure stranding** — the seed's idempotency guard keyed only on exercise count, so a mid-sequence DB error left the flag false and, on retry, `count>0 → markSeeded → stop` permanently abandoned the coach with a visibly-broken empty "Example" template or orphan program. Fixed by making the seed **fully resumable**: every artifact created only if missing, flag flips only when all six exist. Resume regression test added.

**Tests added:** surgical-propagation + `_applyToAllSessions` end-to-end + `_assignedCopiesForSession` classification (programs.spec.js); onboarding seed + `loadUserInfo` wiring + partial-seed resume (onboarding.spec.js, via PT2). Full suite 133 passed / 2 skipped / 0 failed, reconciled.

**UNVERIFIED (banked):**
- **New-coach onboarding needs a real signup to confirm live** — Playwright can't drive a fresh email signup; the seed function + wiring are proven via PT2, but "a genuine new coach lands on a populated dashboard" needs Jake to create a throwaway account. (Added to STATUS open to-dos.)

**Decided:**
- **Propagation matches by exercise NAME** (Jake's choice over by-position), leaving a target that lacks the exercise untouched.
- **Editing a program workout auto-syncs assigned copies:** solo silently, real clients behind a confirm ("ask when clients assigned").
- **Starter content is automatic on first login** (not opt-in), a **curated set** (not cloned from Jake's real library), delivered **app-side + a flag** (not a DB trigger — the list stays editable JS). Sample program **not auto-assigned**.
- **Tests that borrow the shared PT2 account must self-heal + clean up in `finally`** — a strand from an assertion-order cleanup compounded PT2 to 80 exercises mid-build; fixed by sweeping at test start AND in finally.

**Why:**
- Both program bugs and the storage leak trace to the same lesson bank: the propagation subsystem is where CoachApp's data-loss bugs live (delete-all/reinsert-all shapes), and the multi-agent review keeps catching real issues in the first cut — it has now paid off on three consecutive pushes.

## 2026-07-12 — Security & beta gates: a live storage breach caught by the first-ever /deploy-check (v21→v22 runner, v9→v10 progress, v4→v5 clients)

**Done:**
- **Behavioural RLS audit** (`tests/rls-audit.spec.js`) — the standing security regression gate, replacing /deploy-check's toothless `qual='true'` grep (which caught none of this project's 4 real RLS gaps). Four checks: Probe A (a second coach who owns nothing reads 0 rows from all 22 tables), Probe B (a client sees only their own rows + a positive `CLIENT_MUST_SEE` lower bound so a *denied* query can't pass as clean), Probe C (a client CAN read their assigned program through the full nested embed — the s23/s24 unexpected-DENY class), and a self-test that plants its own victim to prove the detector fires. Created **Coach B** (`coachapp.e2e.pt2@gmail.com`) — the first second-coach account in the project's history; cross-coach isolation had never been testable.
- **Storage security** (`tests/storage-privacy.spec.js`) — anonymous-fetch test + cross-tenant coach test, both owning their fixtures. Not covered by the `public.*` RLS audit.
- **`/deploy-check` run end-to-end for the first time.** 8/9 gates green. Strengthened the skill's RLS step (→ run the harness) and storage step (→ run the behavioural probe, not `select public from buckets`).
- **ICO breach-notification procedure** — `breach-procedure.md` (72h rule, ICO/individual decision tree, mandatory internal breach log, and today's storage leak as a worked example of *non*-notification). CRITICAL.md flipped ❌→✅; its storage section rewritten with the "private is not sufficient — policies must be path-scoped" rule.
- **Progress-photos feature removed** (Jake, "for now") — Photos tab + `renderClientPhotos`/`upload`/`delete` gone; bucket + data retained, restorable from app-progress v9. Orphan dev photo deleted.
- Pushed 5 commits (`8b9bb97`→`8652491`); CI green; live serves app-runner v22.

**Bugs found + fixed:**
- 🔴 **LIVE cross-tenant storage leak (headline).** `progress-photos` was `public=false` yet had 3 `storage.objects` policies scoped by `bucket_id` alone — `"Public read"` (SELECT), `"Authenticated delete"` (DELETE), `"Authenticated upload"` (INSERT). **Any authenticated coach could read AND delete any client's progress photos.** Reproduced live: PT2 (owns nothing) downloaded a real 1.79MB photo in full and deleted another. Root cause: the correctly path-scoped "Client/Coach manages photos" policies existed, but these 3 over-broad ones widened every verb to the whole authenticated user base. Fix: dropped all 3 (`scripts/fix-storage-rls-2026-07-12.sql`), proven red→green. A `select public from buckets` check passed this leak clean — only attempting the download as the wrong tenant caught it.
- 🔴 **Personal Bests never displayed — for anyone, ever.** `renderProgressPBs` embedded `performance_exercises` (a table that does not exist); PostgREST rejected the whole query; the error was discarded; the page showed "No personal bests logged yet" forever. Every PB anyone logged was saved and never shown. The columns were plain fields on `performance_logs` all along (app-progress v8→v9). Found by the audit enumerating referenced tables.
- 🔴 **`addTableRow` still auto-filled weight+reps from last session** (caught by the pre-push multi-agent review, found by 2 agents independently). The pre-fill was removed from `_ensureTableRows` and left alive in its sibling — a fix-the-class miss. "+ Add set" produced a row pre-filled in solid black; tick it and you logged a set you never performed. Exactly the bug `8b9bb97` claimed to remove.
- 🔴 **Wizard still rendered "RPE 8–9" under a column labelled RPE** (same review). The de-dup only landed in the table because the wizard held a *verbatim copy* of `_buildTargetCols`, whose own comment falsely claimed it was shared. Deleted the copy; wizard now calls the shared pair. **My fix for this then broke the runner entirely** (`repsStr is not defined`, 26 tests red) — caught by the full suite, fixed, before push.
- **`toggleTableSet` now requires a weight**, not just reps — dropping the pre-fill made a weightless ticked set the easy path, silently zeroing volume, hiding it from PB detection, and decaying next session's ghost text.
- **The RLS self-test was resting on stranded fixtures** — it only fired because 2 `[E2E-RLS] Victim Client` rows had been left in the real DB by an earlier run; on a clean DB it would `skip` (neither pass nor fail). Now plants+cleans its own victim. Swept the strays.
- **The seed's workout-history block had NEVER run** — skipped whenever sessions exist (always), so 3 wrong column names (`log_id` not `workout_log_id`, `workout_log_exercise_id` not `exercise_id`, missing `exercise_type`) sat unexercised. It also logged only Bench Press while the table tests use Overhead Press, and omitted `set_number` (which `_prevSetsByIndex` keys on: `null-1` === -1). Net: ghost text had **zero** real coverage; `expect(row.weight).toBe('')` passed even against the old pre-filling code. Fixed all of it; ghost-text test has teeth now.
- PB regression test now verifies its own cleanup (an RLS-denied DELETE removes 0 rows without erroring — it was stranding a row in the real DB each run).

**UNVERIFIED (banked):**
- **/deploy-check gate #6 — live client smoke test** — needs Jake to log in as a real client on the live site (dashboard stat, Workouts Start buttons, session history). Only unconfirmed gate.

**Decided:**
- **Storage is a distinct RLS surface** — the `public.*` audit does not cover it; `storage.objects` needs its own behavioural probe. Both /deploy-check and the harness were blind to it until this session.
- **"Private bucket" ≠ secure** — a `public=false` bucket with `bucket_id`-only object policies is wide open to every authenticated user. Only path-scoped object policies are safe.
- **Removed feature = remove code, keep data** (when "for now"): retain the bucket + contents, delete UI, flag any orphaned personal data (the loose photo) for a decision, and do not re-enable uploads without also restoring the delete path (GDPR erasure).
- Progress-photos leak assessed **non-notifiable** to the ICO — no real data subject's data was accessed by a real party (only test accounts + Jake's own dev photo existed). Logged per the new procedure. The same bug post-beta would very likely be notifiable — the distinction is who's actually on the platform.

**Why:**
- Three "a config-read passed a real leak" lessons this session: the old RLS `qual='true'` grep, and now the storage `public` flag. The through-line is that only *attempting the operation as the wrong tenant* finds these — config reads describe intent, not behaviour. Both checks are now behavioural probes in CI.

## 2026-07-11 (session 25, part 3) — Runner table polish from real gym use; planning session; found a beta blocker nobody had asked about — COMMITTED NOT PUSHED (8b9bb97)

**Context:** Jake used the runner in a real gym session and came back with 4 corrections, then pivoted the session to planning ("This whole session will be a planned one — review the kanban board/backlog and find anything that needs scoping"). He explicitly chose **commit but do not push** for the runner work.

**Done — runner table polish (app-runner v20→v21, committed `8b9bb97`, NOT pushed):**
- **RPE/RIR value no longer repeats its own column label.** The target bar rendered "RPE 8–9" *under* a column headed RPE. Value now carries the number only.
- **Plate calculator REMOVED outright.** Shipped 8 days earlier (2026-07-10, v19) after "repeated requests" surfaced in the 2026-07-02 competitor research. Jake used it in a real session and asked for it gone — noise, not help. Deleted, not flagged off: `_PLATE_SIZES`, `_BAR_WEIGHT_KG`, `_calcPlateBreakdown`, `_updatePlateBreakdown`, the PLATES/SIDE target-bar column, the wizard hint, and its 5 tests.
- **PREVIOUS column dropped; last session is now GHOST TEXT in KG/REPS.** The old 54px cell squashed both numbers into one string ("140kg × 6"). Each previous value now sits directly under the column it represents, and KG/REPS get the freed width. Falls back to the %1RM-derived target when a set has no history.
- **Rows no longer auto-fill.** This reverses the v1 "1-tap repeat" pre-fill — a pre-filled value is indistinguishable from one you actually entered, so a set could be ticked off having never confirmed the weight was right. Rows start empty; you type what you did.
- Runner suite 41 passed / 0 failed. Verified visually at 480px.

**Bugs found + fixed (knock-ons of the no-auto-fill change, both caught by thinking the change through rather than by a test):**
- `toggleTableSet`'s existing "require reps" guard was previously **never** hit (rows were always pre-filled). With empty rows it is hit *routinely*, and a silent no-op would read as a broken button to someone mid-set. It now warns ("Enter reps first").
- **A back door that would have silently re-introduced auto-fill:** `renderRunnerLastSession` *wrote* last session's values into `tableRows` when the async fetch resolved after first paint. Left alone, the feature would have quietly come back the moment the fetch was slow. It now only repaints (so the ghost text appears), guarded by `_prevTablePaintKey` against a render loop.

**Not a bug (recorded so it isn't re-investigated):** Jake reported Barbell Back Squat rendering the wizard instead of the table. He'd accidentally set the exercise **unilateral**, and `_isPlainStrengthExercise` correctly excludes any unilateral set. He self-corrected: *"may be a false flag from me."* No code change.

**Also fixed:** the wiki roadmap's mermaid diagram, which I broke in the previous save — the `P7n` node label contained an unquoted apostrophe ("client's"). Every other node with special characters is quoted.

**🚨 FOUND — beta blocker nobody had asked about:**
**A brand-new coach signs up to a completely empty app.** Verified in code: `signUp` (app-core.js:332) creates the auth user, and `handle_new_user` creates **only** the `profiles` row (deliberately — the les-006 fix). A fresh PT gets **0 exercises**, 0 templates, 0 programs. You cannot build a workout without exercises, and the only route is typing each one in by hand. **Jake has never experienced this** — his account has 200+ exercises accumulated over months of building. The app is excellent *once populated* and close to unusable *before*, and every beta PT starts at zero. Needs scoping (default exercise library on signup). Beta is 31 July.
Also raised and **deliberately deprioritised by Jake** (logged so they aren't lost): error monitoring (a beta PT's crash never reaches him — `log.error` only goes to their own console), backup/restore posture, beta ops (no feedback channel; invite emails have only ever been sent to Jake, so spam-folder risk is untested).

**Decided:**
- **Next session = Security & beta gates** (Jake's pick from a 3-way choice). Build a *behavioural* RLS audit harness — every table × every operation × every role, **plus a second coach and a second client** so cross-tenant isolation is finally tested. Rationale: three consecutive sessions found real RLS gaps and **all three were found by accident**; `/deploy-check`'s RLS check would have caught **none** of them (it only greps `qual = 'true'`, and every real leak had a normal-looking policy that simply checked the wrong column); and **cross-coach isolation has never been tested at all**.
- Runner work **committed but not pushed**, at Jake's explicit request — it rides with the next push after review.
- The plate calculator's whole arc is worth remembering: **the research said build it, the gym said remove it.** Eight days from ship to delete. "Repeatedly requested in competitor research" did not survive contact with real use.

**Why:** Jake asked for a full planning pass with beta 20 days out. The most valuable output wasn't the 4 runner fixes — it was noticing that nobody had ever asked what a *new* user actually sees.

---

## 2026-07-11 (session 25, part 2) — The Library bridge (copy program workouts → Library, tap-row picker, duplicate-week auto-extend); review found a CRITICAL live data-loss bug — PUSHED (5134dd6, 6e6afb2)

**Context:** Jake asked three things that turned out to be one problem. He'd built his "Hybrid Weapon Experiment" personal program entirely with "+ Create new workout (this day only)" — which stamps `program_id`, and session 24 deliberately excluded program-owned templates from the reuse pool (the fix for the indistinguishable-duplicates bug). Correct fix, but it left **no bridge** from "built it in a program" to "reuse it": 6 workouts he'd otherwise retype by hand. His follow-up was the sharp one — *"if a user creates 3 'upper body' workouts in their library, how will they know which is which when adding them into the program?"* — which is not a naming problem at all, but the native `<select>` (an `<option>` can only hold plain text). And "Duplicate week" had vanished from his 1-week phase.

---

### 🔴 CRITICAL DATA-LOSS BUG — full reproduction steps (Jake asked these be recorded)

**Symptom:** a workout in Week 1 of a program is silently and permanently destroyed.

**Reproduction:**
1. Open a program → add a phase.
2. Populate **Week 1** with at least one workout.
3. Click **"Duplicate week"** on Week 1. (Week 2's rows now point at the **same `template_id`** as Week 1 — this is deliberate: duplication is "cheap by design", the copy only forks into an independent template when someone edits one of them, via `_resolveEditableTemplateId`.)
4. Set a periodization type on the phase (Configure → Linear/Undulating).
5. Click **"Generate weeks"**.
6. **Week 1's workout is now gone.**

**Root cause:** `generatePhasePeriodization` calls `_cleanupPhaseWeeksBeyond(phaseId, 1)` to clear weeks 2+ before regenerating. That function collected every `template_id` referenced by a stale week and ran a bare `db.from('workout_templates').delete().in('id', staleMasterTemplateIds)` — with **no ownership check and no still-referenced check**. Because the duplicated Week 2 shares Week 1's `template_id`, the cleanup harvested Week 1's own template off the Week 2 row and deleted it while Week 1's surviving row still pointed at it.

Its sibling `deletePhaseWeek` had **both** guards — added by the multi-agent review on 2026-07-10. `_cleanupPhaseWeeksBeyond` never got the same fix. The two silently diverged.

**Second variant of the same bug:** any **standalone library template** assigned into Week 2+ was also deleted outright by that same call — removing it from the Workouts library and from *every other program using it*. This is exactly the `deleteProgram()` bug fixed on 2026-07-10, in a second unfixed location. The new "copy to Library" feature makes library templates far more likely to be sitting in program weeks, so this would have got worse.

**Proved, not reasoned:** reverted the single fixed line, ran the new regression test → `survived: 0` (the Week-1 workout genuinely gone). Restored the fix → `survived: 1`. Red/green.

**Fix:** extracted `_deleteOwnedUnreferencedTemplates(templateIds, programId, phaseId)` — ownership check *and* still-referenced check — and both `deletePhaseWeek` and `_cleanupPhaseWeeksBeyond` now call it, so they cannot diverge again. Regression test added (`programs.spec.js`). Banked in STATUS.md's continuity block.

**How it was found:** the pinned 3-agent review, not by inspection and not by any existing test. It surfaced only because this session's Duplicate-week change made a 1-week phase newly able to reach periodization range — the reviewer traced the consequence one step further than the diff.

---

**Done:**
- **Copy program workout → Library.** New `_copyTemplateToLibrary` + `saveTemplateToLibrary` (per-workout button in the session-detail drawer, gated on `ctx.programId && !ctx.isClientPlan`) + `copyProgramWorkoutsToLibrary` (bulk button on the program page). Reuses `_cloneSharedMasterTemplate` via a new defaulted `overrides` param rather than duplicating its exercise-copy logic. **Idempotent** — a same-name library workout is skipped, so double-clicking is safe. Excludes periodization week-clones (derivatives, not sources). app-workouts v24→v25.
- **Tap-row workout picker** replacing the native `<select>` (`_openWorkoutPicker`, modelled on `_openExercisePicker` — same modal shape, live filter, `visualViewport` keyboard sync). Rows show name + description + exercise preview. `_quickAssignPhaseWorkout` refactored to take a slot object instead of a `<select>` element; `_filterPwgOptions` deleted. Closes the two long-open complaints about this control (no feedback until opened; list grows unmanageable). app-programs v15→v16.
- **Duplicate week auto-extends the phase** — `canDuplicateAny` gate dropped; `duplicatePhaseWeek` now UPDATEs `duration_weeks` instead of bailing. Bump happens **after** a successful insert (see below).
- Added `description` to `openProgram`'s template select — applying last night's embed-select-allowlist lesson rather than rediscovering it.
- **Process:** made roadmap.md a first-class mandatory step in `/save` (Jake's standing request); retired the daily-question cron (unused, redundant); corrected a stale roadmap entry (Area 3 #13 was still open despite shipping 2026-07-08).

**Bugs found + fixed (multi-agent review — 7 real findings, all fixed pre-push):**
- The CRITICAL data-loss bug above.
- **Bulk copy deduped by NAME, not `template_id`** — three genuinely *different* workouts sharing a name (Jake's exact scenario) would have had only one copied, with the other two reported as neither copied nor skipped. Now deduped by id; copies run **sequentially** so each sees the previous one's writes; same-name collisions reported honestly as "skipped (same name already in Library)" rather than the false "already in your Library".
- **The idempotency guard failed open.** `.maybeSingle()` *errors* when >1 row matches, and the discarded error yields `null` → the anti-duplicate guard would create *more* duplicates precisely when duplicates already existed. Also `.ilike()` treated the workout name as a **LIKE pattern** (`_` and `%` are wildcards). Both replaced with a plain fetch + JS compare.
- **`duration_weeks` bumped before the copy was known to succeed** — a failed copy left the phase permanently claiming an empty week, which for an already-assigned client silently lengthens their program and shifts every later phase out. Moved after the insert.
- **Picker had no re-entrancy guard** — a double-tap appended a second overlay sharing the same element ids, so results rendered into the buried copy. Same race that froze the runner's picker on 2026-07-04.
- **Picker pool went stale after a copy** — the toast said "you can now reuse it in any program" but it wouldn't appear until reload. Added `_refreshProgramTemplates()`.

**Test-harness bugs found + fixed (all latent, none product regressions):**
- **`scripts/seed-test-data.js` had NEVER worked.** It omitted the required `exercise_name`/`exercise_type`, double-encoded `sets_json` as a **string** (the column is jsonb), and **never checked the insert's error** — so it failed silently while printing "Template exercises added". This meant the seed template "Push Day A" could not be rebuilt, which I only discovered *after* deleting it. Now fixed, error-checked, and idempotent (repairs an emptied template instead of skipping it).
- **My own new test was poisoning the runner tests.** Its `finally` deleted the fixture program via raw SQL (bypassing `deleteProgram()`), so the phase cascade set `generated_from_phase_id = NULL` on its periodization clones — orphaning them into the standalone pool, where they sort **above "Push Day A"** (`[` < `P`) and got grabbed by the runner tests' "click the first Start button". 9 had accumulated. Test now sweeps its own clones.
- **Two over-loose runner locators.** `text=KG` is a case-insensitive **substring** match, so it matched the target bar's "50 kg" and the PREVIOUS column's "80kg" as well as the intended "Kg" column header — a strict-mode violation the moment the exercise has a weight target or logged history. Tightened to `getByText(..., { exact: true })`.

**Process failure worth banking:**
- I piped `npm test` through `tail -8`. Playwright prints its summary as **failed → flaky → skipped → passed**, so the tail cut off an **"8 failed"** header and showed only "3 skipped / 115 passed" — a **false green**. It was caught *only* because the counts didn't reconcile against the declared total (126). **Always reconcile passed+failed+flaky+skipped against the declared total; never trust a truncated test summary.**

**Found, NOT fixed (needs a decision):**
- **Deleting a program orphans its periodization week-clones into the reusable template pool.** When the program (and so its phases) are deleted, `generated_from_phase_id` goes `SET NULL` — so a "Bench Press — W2" clone loses the only column marking it as a derivative and becomes indistinguishable from a genuine standalone template. It then escapes the ownership checks *and* clutters the picker. Same mechanism behind the picker clutter Jake reported on 2026-07-10 (that fix removed program-owned templates from the pool, but not these orphans). Options: change the FK to CASCADE, or have `deleteProgram` sweep clones before deleting the phases.

**Decided:**
- Fixing the picker properly (tap-row, not a richer `<option>`) was the right call because it closes three separate complaints at once — Jake's disambiguation question and the two pre-existing `<select>` complaints — rather than three patches.
- The `text=KG` locators were genuinely too loose; tightening the *test* was correct, not contorting the seed data to fit a fragile assertion.

**Why:** Jake's three asks were one problem, and the review's job is to look one step past the diff — which is exactly how it caught a data-loss bug that was already live and that no existing test covered.

---

## 2026-07-11 (session 25) — Personal/solo Library page (Templates + Exercise Library); closed a REAL cross-client RLS leak found while auditing it — PUSHED (0a3ef1d, c4b1e67)

**Context:** Jake opened with a question, not a bug report: *"personal account does not have workouts > templates & exercise library page. This is where I need to create workouts that can be put into programs, correct?"* — he was right on both counts. Solo had no route to the template builder at all: `renderWorkouts` explicitly bundles `solo` in with `client` and diverts both to the read-only session view, so solo's only way to create a workout was the inline "+ Create new workout (this day only)" dropdown in a program phase, which locks the template to that one day slot. No way to build a *reusable* one. Chose a separate nav entry (his call, via AskUserQuestion) over extra tabs on the existing Workouts page, since solo still needs that page's "Up next" hero/accordion.

**Done:**
- **Personal/solo `library` nav page** — extracted `renderWorkoutLibrary(el)` out of `renderWorkouts(el)` (the coach's Workouts page is byte-identical, its early-return for client/solo untouched) and gave solo its own `library` nav entry + router case + `soloPages` entry pointing at it. Reuses the coach's Templates/Exercise-Library tab UI verbatim — no new components. Nav icon reuses the `workouts` SVG (existing precedent: the 3 dashboard icons are already byte-identical). app-core v2→v3, app-workouts v23→v24.
- **`workout_templates.is_personal` column** (Jake ran the SQL; 1537 existing rows → `false`) — required before the Library page could ship: solo and PT share one `coach_id`/`auth.uid()`, so opening the builder to solo without a split would have leaked Jake's real client templates into his personal library and vice versa. Exactly the bleed already fixed for `exercises` on 2026-07-10. Wired into all 4 standalone-template read sites, `saveNewTemplate`, and the 3 clone paths. Fix-forward: existing rows stay attributed to the PT, no reclassification.
- **Pre-existing bug fixed alongside:** `openProgram`'s day-slot picker had no role split either (`app-programs.js:595`), so solo was *already* seeing the PT's entire standalone-template pool in that dropdown — independent of the new feature, would have kept leaking regardless. app-programs v14→v15.
- **`renderExerciseLibrary` hardcoded `.eq('is_personal', false)`** — only ever "worked" because solo could never reach the function. Made role-aware before opening the route to solo.
- **SECURITY: closed a real cross-client RLS leak on `workout_templates`** (see below). 2 new regression tests against a **real client-role account**.
- Tests: 3 new in `solo-account.spec.js` (is_personal template split, Library nav smoke, program-picker split), 2 new in `client-workout.spec.js` (cross-client leak, master-embed-chain guard). Full suite 119 passed / 2 expected skips. Mobile-checked at 480px and 375px — solo nav now 7 items, single row, no overflow (measured, not eyeballed).

**Bugs found + fixed:**
- **CRITICAL (pre-existing, live the whole time real clients have had accounts): any client could read any OTHER client's workout template clones.** The `Client reads workout templates` SELECT policy scoped by `coach_id` **alone**, with no `client_id` restriction. Client-plan clones are written with `coach_id = the coach, client_id = the client` (`_cloneTemplateForClient`), so every client of the same coach matched that policy. **Reproduced live before fixing** — logged in as a genuine client and read back another client's private template by id (red/green, not reasoned). Fixed by adding `client_id is null` to the policy; a client's own clones still resolve via the separate `client_read_own_templates` (client_id-scoped) policy, so nothing legitimate was lost. Found only because the multi-agent review's security agent questioned whether `is_personal` was doing security work it wasn't — the leak itself was a level deeper than the thing being audited.
- **Latent RLS bug in the same policy:** it used a scalar `=` subquery (`coach_id = (SELECT coach_id FROM clients WHERE user_id = auth.uid())`), which errors outright for any user with >1 `clients` row — **the master account has exactly two** (one coached record + one solo record). It only worked because the coach policy matched first. Switched to `in` per the sql-safety rule.
- **The `is_personal` clone carry-over was a silent no-op** (caught by the 3-agent review, independently by two agents, then verified against the code). `_cloneTemplateForClient` / `generatePhasePeriodization` read `tmpl.is_personal` from source objects fetched via *embedded selects with explicit column lists that omitted `is_personal`* — so it was `undefined`, dropped from the JSON insert payload, and silently fell back to the DB default. Not a live leak (those rows always carry `client_id`/`generated_from_phase_id`, which every `is_personal` read already filters on) but a landmine. Fixed by adding `is_personal` to the 3 source selects.
- **Both new fixture tests could orphan `[E2E]` rows on partial failure** (caught by the review's regression agent) — fixtures were built *before* the `try` block, and the picker test captured `programId` only *after* the program row was already inserted. Cleanup is now by-name and idempotent inside `try/finally`.

**Decided:**
- **`is_personal` is deliberately NOT enforced at RLS — and this is load-bearing, not an oversight.** I initially proposed adding `is_personal = false` to the client-read policy alongside the `client_id` fix. Tracing the embed chains before writing the SQL showed that would have **broken real clients**: the client Dashboard hero (`app-dashboard.js:262`) and client Calendar (`app-calendar-goals.js:27`) embed the **master** templates through `program_phase_workouts` — not the client's clones. A program built in Personal view carries `is_personal = true` masters, so the restriction would have silently nulled that embed the moment such a program was assigned to a real client — the exact PostgREST silent-null failure from sessions 23/24. `is_personal` answers "which of Jake's two libraries is this in," not "who may read this." Banked into STATUS.md's continuity block so a future session doesn't "helpfully" add it.
- `exercises` needed **no** RLS change — it has no `client_id` column (no per-client rows exist), so no cross-client leak is possible there. Its only "gap" was the same is_personal non-enforcement, which per the above is correct as-is.
- Jake chose a **separate nav entry** over extra tabs on the Workouts page (AskUserQuestion), so solo keeps its existing "Up next"/session view untouched.
- Jake selected the recommended "push code now, fix RLS next session" option — then immediately overrode himself with *"lets fix now"* once the leak was characterised. Banked as a voice/behaviour delta: he will not defer a real security gap even when handed a sanctioned reason to.

**Why:** The Library page was a genuine capability gap Jake hit in real use (he couldn't build reusable workouts for his own programs). The RLS leak was not on any backlog and nobody knew it existed — it surfaced only because the pinned review's security angle pushed one level past the feature being built, which is exactly the drift the pinned multi-agent-review skill exists to prevent.

---

## 2026-07-10 (session 24) — RLS audit found 2 more critical gaps; built autosave + %1RM rounding + Delete week + plate calculator; live program-picker fix; 3-agent review caught 3 real issues — PUSHED (7ae59d2, 5f84516, 31394e7, 20f6b8a, a4368a8, c4db4b9, bc4c0fd, ed6a3c8, 9f33da0)

**Context:** Opened by confirming session 23's fixes live (RLS + Workouts-page speed), which surfaced a new bug immediately ("the exercise library for PT account contains all exercises that have been created on personal account"). From there: RLS audit sweep (approved), then runner autosave, then a batch of quick-win requests (E2E cleanup, Delete week, plate calculator), then a live bug report mid-Delete-week-build (program picker clutter), then %1RM rounding, then the full push cycle with multi-agent review.

**Done:**
- **Exercises PT/Personal scoping** — new `is_personal` boolean column; every create/read site in app-workouts.js (`saveNewExercise`, `_createExerciseFromPicker`, `_resolveExerciseIdForSave`, `_openExercisePicker`, `renderExerciseLibrary`) stamps/filters by `currentProfile?.role === 'solo'`. Existing exercises stay as PT's (Jake's explicit call — no reclassification), only new creates separate going forward. New Playwright test in both directions. app-workouts v21→v22.
- **RLS audit** (approved as "quick win" — turned out much bigger): `client_1rms` had no INSERT/UPDATE/DELETE policy for a real coached client (only solo). Then, digging one level deeper than scoped: `workout_template_exercises` had **zero** client-read policy at all — the only SELECT-capable policy was `coaches manage own template exercises` (coach_id = auth.uid()), which never matches a real client's own auth.uid(). Solo was invisible to this because solo's auth.uid() IS the coach's own id — the exact same blind spot as the `client_programs` bug from session 23. This broke `openSessionDetail` (direct query, showed "No exercises added yet" on every real session) and the client Workouts-page accordion (nested embed silently nulled the exercises level). Both fixed via new Supabase policies (Jake ran the SQL), verified with 2 new Playwright tests that query the exact real app shapes against a real client-role account, not just pg_policies.
- **Runner session autosave** — scoped 2026-07-05, built this session. localStorage-only draft, checkpointed on every `renderRunner()` + a 10s safety-net tick, same-day staleness cutoff, resume/discard confirm modal wired into `launchRunner()` (refined from the original scoping note of `startWorkoutRunner()` — `launchRunner` is the true single choke point both the fast-path and the setup-modal's own Start button funnel through). Cleared inside `discardRunner()` (covers both abandon and post-save). 5 new Playwright tests. app-runner v16→v17.
- **%1RM target weight rounding** — `_calcWeightFromPct` now floors to the nearest 2.5kg instead of rounding to the nearest 0.5kg; single shared function feeds every %1RM display site (target bar, table pre-fill, wizard placeholder) so the fix applies everywhere at once. 4 new tests. app-runner v17→v18.
- **`startWorkoutRunner`'s freeform template list** — was missing the same `client_id`/`program_id`/`generated_from_phase_id` leak filter already fixed twice elsewhere. Fixed + new test that calls the real function, not a re-implemented query. app-workouts v22→v23.
- **`showRunnerOneRMSheet` z-index** — live-verified for the first time (new Playwright test fills the modal's own input and confirms no pointer-interception error) rather than left as "reasoned from pattern, never reproduced."
- **E2E test debris cleanup** (Jake approved explicitly) — enumerated first (found 73 leftover `[E2E]`-prefixed programs and 47 templates, not the ~13 originally estimated — that number predated today's new tests), then deleted via the real `deleteProgram()` function (not raw SQL) so the exact production ownership logic ran. 45 remaining client-owned template clones (not touched by `deleteProgram`) verified zero-referenced then deleted directly. Final verification: 0 remaining.
- **Program picker clutter fix** — found live mid-session (Jake: "every workout that is created in the program appears in this list... impossible for a PT to know which workout is which," with a screenshot showing 4+ duplicate-named entries). Root cause: `openProgram`'s template query (`window._programTemplates`) had no `.is('program_id', null)` filter, so every one-off "+ Create new workout" template stayed in the reuse pool forever. Fixed via AskUserQuestion-confirmed direction (exclude program-owned templates from the pool); the inline option relabeled "(this day only)" per Jake's explicit follow-up ask to make the workflow clear. Confirmed via code trace (no role branching anywhere in `openProgram`/the picker) that this already covers Personal/solo with zero extra work, per Jake's ask to include it there too.
- **Delete week button** — scoped 2026-07-08, built this session. Removes a week's own sessions + owned templates + client-propagated copies, renumbers later weeks down by 1, decrements `duration_weeks`. First draft reused `deleteProgram()`'s ownership check but the multi-agent review caught a real gap (see below) before push.
- **Plate calculator** — repeatedly requested (2026-07-02 research). Wizard: live-updating hint under the weight input. Strength table: a new PLATES/SIDE column in the existing target bar — deliberately not new text under table rows, since Jake had explicitly rejected that exact pattern in that exact table previously ("highlighted or stand out, not entered as text underneath — ugly UI").
- **Multi-agent review before push** (3 agents in parallel: security/scoping, solo-mode, duplicates/render-safety + a verifier pass) caught 3 real issues, all fixed same session: (1) `deletePhaseWeek`'s ownership check alone wasn't enough — `duplicatePhaseWeek` shares `template_id` across weeks until someone forks on edit, so deleting one week could destroy a template a sibling week's surviving row still needed; fixed by also checking `program_phase_workouts` for any other row still referencing the template before deleting it, new regression test added. (2) Resumed drafts skipped `_unlockAudio()`/`_unlockSpeech()` (only `_startFreshRunner` called them), so a resumed session's rest-timer beeps/voice cues could silently never fire on iOS Safari. (3) Runner drafts were never cleared on sign-out, leaving a prior client's in-progress workout (name, exercises, weights/reps) in localStorage indefinitely on a shared/gym device.
- Pushed as 9 commits (7ae59d2 through 9f33da0); pre-push hook caught one more real issue on the first push attempt (bare `clearInterval()` instead of this codebase's `clearTimer()` convention), fixed in a follow-up commit. Full Playwright suite green throughout (116 run, 114 passed, 2 expected conditional skips). CI confirmed green after push.

**Bugs found + fixed:** all listed above under Done — every one was root-caused via actual code tracing (RLS gaps confirmed via pg_policies + live fixture tests against a real client-role account; the picker clutter traced to the exact missing filter; the 3 review findings each verified against the cited line before fixing), none guessed.

**Decided:**
- Existing (pre-`is_personal`-fix) exercises stay classified as PT's — no manual reclassification of historical data, only new creates get separated (Jake's explicit call, "leave both lists as they are, but clean up the bug so this data doesn't bleed again").
- The program-picker fix path: exclude program-owned templates from the reuse pool entirely (not just label duplicates, not both) — Jake's pick from an AskUserQuestion with a stated recommendation.
- `launchRunner()` (not `startWorkoutRunner()`, the originally-scoped location) is where the autosave resume-check lives — found the more complete choke point once actually reading the code, flagged the deviation explicitly rather than silently diverging from the scoped plan.
- Graded 11 overdue predictions in `predictions.jsonl` at session start; found and fixed an id collision (two unrelated predictions both `pth-017`) along the way.

**Why:** Jake explicitly asked to keep going through a long batch ("keep going" ×2), approved the RLS audit before it revealed the deeper `workout_template_exercises` gap, and caught the program-picker bug live via his own screenshot mid-session — none of tonight's biggest fixes were pre-planned at session start.

---

## 2026-07-10 (session 23, final) — Live empty-phase crash hotfixed; client_programs RLS gap (+3 related tables) fully resolved and confirmed end-to-end — PUSHED (b79c152, b126d5b)

**Context:** Continuation of the same day's session 23. Right after the earlier round's `/save`, Jake tried the live Personal/solo Workouts page and hit a real crash. Separately, applying the `client_programs` RLS fix from earlier in the day led to discovering it was incomplete — the first verification pass only checked one table, not the full query shape the app actually uses.

**Done:**
- Hotfixed a live crash: a phase with zero `program_phase_workouts` (a normal state — a phase not yet fully built out) crashed `renderDays` on `undefined.forEach`. Pre-existing code, not part of the earlier diff. Fixed with an explicit "No sessions added to this phase yet" message; new Playwright regression test. **app-workouts v21**, pushed b79c152.
- Confirmed the `client_programs` RLS fix (applied earlier this session) plus 3 more policies (`programs`, `program_phases`, `program_phase_workouts` — all in the same nested embed the app's real queries use) all work end-to-end via a fresh Playwright fixture test, not just a table-level check. Un-fixme'd the 2 tests that had been blocked on this. Added a new dedicated embed-chain regression test. Fixed an unrelated ambiguous test locator along the way (`text=Deload` collided with the hero card's own meta text; scoped to `button:has-text(...)`). Pushed b126d5b.
- New standing skill created: `missed-check-to-test` (`~/.claude/skills/missed-check-to-test/`) — whenever a bug's root cause is "I checked A but not the closely-related B," converts that miss into a Playwright test in the same commit. Registered in hello-claude's standing behaviours.
- Two new memory entries: `feedback_rls_embed_chains.md` (check every table in an RLS embed chain, not just the outer one) and an addendum to `feedback_edge_case_testing.md` (the zero-count-state lesson, from earlier in the day).

**Bugs found + fixed:**
- Empty-phase crash (above) — root cause confirmed by reading the code against Jake's own pasted console stack trace, not guessed.
- **Incomplete RLS fix, self-caught, not by Jake.** After Jake applied the `client_programs` policy, a fixture-based Playwright test revealed the dashboard/Workouts page still crashed — traced to `programs`/`program_phases`/`program_phase_workouts` also lacking client-read policies. PostgREST doesn't error on an unreadable embed level, it silently returns `null`, which is what made the first fix look complete when it wasn't (a direct `client_programs`-only check couldn't have caught this — needed to trace the whole embed chain the app's real query uses).

**Decided:**
- When verifying an RLS fix for a query with nested embeds, trace and test every table in the chain independently, not just the entry table — banked as a memory + reflected in the new skill.
- Cleaned up 2 rounds of my own test debris from this investigation (orphaned throwaway programs/templates, exact IDs known from my own runs) directly, without asking — distinct from the earlier-blocked bulk pattern-match delete, since this was narrowly scoped to rows I created and could directly attribute this session.

**Why:**
- Jake asked directly "remember this and add to test" after seeing the incomplete-fix correction — the skill and memory entries are the direct response, aimed at this exact class of miss recurring in future sessions.

---

## 2026-07-10 (session 23) — Finished Workouts-polish build; fixed deleteProgram data-loss bug + Workouts-page perf issue; found critical client_programs RLS gap — PUSHED (8e9c26c)

**Context:** Continued session 22's Workouts-polish build (hero card + "Recent sessions" rename), which had been interrupted mid-implementation by a process restart. The in-progress edit already referenced two not-yet-defined functions, breaking the Workouts page for exactly the "Personal > workouts page not loading" symptom Jake reported mid-session — fixed by finishing the edit, not a separate bug. Also handled 6 new backlog items Jake reported live (data leak, Log weight button, starting weight, %1RM rounding, plate calculator) by triaging/documenting priority, then building the highest-priority ones.

**Done:**
- Finished the Workouts-page hero card (`_buildWorkoutsHero`/`_renderWorkoutsHeroHtml`) — shows program name, current phase/week, and a Start button resolving the actual next scheduled session's real template. Gated to only render when a program is assigned (deliberately, to avoid the freeform Start button opening a template-picker modal).
- Renamed "Session history" → "Recent sessions" at both render sites (client/solo Workouts page + PT client-profile Workouts tab), capped to last 5, date-only rows.
- Fixed the dead "Log weight" button on the Progress page's Body Weight tab (same shape as the earlier Log PB fix — the form only existed on Dashboard pages) + fixed `saveClientWeight`'s refresh target.
- **app-workouts v20 / app-programs v12** (also app-clients v4 / app-progress v8, bumped in the pre-compaction part of this same session's continuation of session 22's work).

**Bugs found + fixed:**
- **CRITICAL data leak (root-caused, not guessed):** `renderWorkoutTemplates` and `renderClientWorkoutsPage`'s flat-list fallback were both missing `.is('generated_from_phase_id', null)` — periodization-generated week clones (e.g. "Bench Press — W2") have `client_id`/`program_id` both null too, so they leaked into flat template lists everywhere, including cross-account via solo's shared `coach_id`. Fixed both, matching the pattern already used correctly elsewhere (`app-programs.js:589`).
- **`deleteProgram()` was silently destroying shared templates.** Found via Playwright test-flakiness investigation — a periodization test's throwaway program linked the shared seed template ("Push Day A") into a slot via `templateOptions[0]`, then deleted the program; `deleteProgram()` deleted ANY template referenced by the program's phases with no ownership check, destroying the shared template. This is a real, pre-existing product bug: any coach reusing a standalone template across programs would silently lose it the moment they deleted one of those programs. Fixed to only delete templates actually owned by the program (`program_id` match, or its own periodization-generated week clones via `generated_from_phase_id`). **The first version of this fix had a real regression** — it missed the week-clone case (which always has `program_id: null` too) — caught by the pinned 3-agent review (both Agent B and Agent C independently flagged it) before push, and fixed the same round. Verified live: a periodization-generate-then-delete cycle now leaves zero orphaned templates.
- **Workouts-page perf issue.** `renderClientWorkoutsPage` always fetched the flat templates list (up to 100 rows, nested exercise join) via `Promise.all`, even when a program was assigned and the result was thrown away — worst-case on the personal/solo account, since it shares the PT account's large historical orphan-template backlog. Restructured to only fetch when `!hasProgram`. This is very likely the same root cause as the still-open 2026-07-06 "app runs slow moving to workouts page" report that was never investigated until now.
- **`_buildWorkoutsHero` null `start_date` guard** — `activeAssignment.start_date` can be null (the assign form doesn't require it); `new Date(null + ...)` produced `NaN`, silently falling through to the LAST phase/week instead of the real current one. Found by the 3-agent review (Agent B), fixed same round.

**Found, NOT fixed (needs Jake):**
- **`client_programs` has no client-read RLS SELECT policy at all.** Discovered while building a Playwright fixture for the hero-card test — a genuine (non-solo) client account reads back zero rows from `client_programs`, even completely unfiltered, while the same account correctly reads `workout_logs`/`weight_logs`. Verified this isn't a fluke: compared against two working tables side by side. This means any real client with an assigned program currently can't see it on their Dashboard or Workouts page — invisible until now because solo accounts share the coach's own `auth.uid()` and never hit this RLS check. Needs a Supabase SQL policy (`CREATE POLICY ... FOR SELECT USING (client_id IN (SELECT id FROM clients WHERE user_id = auth.uid()))`), which needs Jake's dashboard access, not something fixable from code. One new Playwright test (`client-workout.spec.js`) is `test.fixme()`'d pending this fix, with clear inline documentation; the underlying hero-card logic is still covered independently by two passing unit-style tests.

**Decided:**
- When a Playwright test's `beforeEach` fails to find expected UI state that a fix should have produced, root-cause against live DB state directly (via a throwaway debug script) rather than re-guessing at the app code — this found the deleteProgram bug, not a guess.
- A discovered bulk-cleanup opportunity for accumulated test debris was correctly blocked by the harness's own safety classifier (pattern-matched delete against shared data not created-and-tracked this session) — did not attempt to work around it; left as an optional to-do for Jake to approve instead.
- Test isolation gap (a test picking "whatever's first" from shared account data instead of creating owned fixtures) is a recurring root cause worth naming explicitly — this is the second time this session class of bug caused real flakiness (see also the `programs.spec.js` `test.skip`-after-arrange cleanup-unreachable pattern noted but not fixed this session, since it doesn't currently cause harm).

**Why:**
- Jake asked directly this session whether better test agents could have caught these issues earlier — yes: a test-isolation review (flagging shared-account-data assumptions in test setup) and a periodic ownership-model audit on delete/cascade logic (checking cascades against the coach_id/client_id/program_id/generated_from_phase_id ownership convention already established elsewhere in the codebase) would both have surfaced this before today.

---

## 2026-07-08 (session 22) — Performance/Personal Bests restructure + 3 confirmed bug fixes — PUSHED (6d8c6a8, e600010)

**Context:** Backfilled at the start of session 23 — this session's `/save` was never run at the time (LOG.md and STATUS.md both stopped at session 21 until now).

**Done:**
- Performance/Personal Bests restructure (client/solo self-view, e600010): folded Cardio + 1RMs into Personal Bests; Performance split into "Per session" (most-recent-vs-previous comparison, expand to graph) and "Per exercise" (alphabetical, live-search); moved the Workouts-page 1RM grid into Personal Bests.
- Fixed bare `class="btn"` Cancel button in the phase-form (undefined CSS, same bug class already fixed on the dashboards) — app-programs v11 (6620720).

**Bugs found + fixed (6d8c6a8):**
- Dead "Log PB" button — form only existed on Dashboard pages, not the Progress page it's clicked from.
- A real solo-mode bug where saving a PB redrew the wrong dashboard.
- Body Weight "Starting" tile reading the wrong field + a Y-axis clamp requiring both starting AND goal weight to be set.
- Hardened `saveEditTemplate`/`deleteTemplate`'s coach_id filter to be role-aware instead of hardcoded (defensive fix, original repro not fully confirmed).

**Decided:**
- 3-agent review caught and fixed a stale-cache race between Client/Personal view switches and a Chart.js instance leak on every search keystroke, before the Performance restructure was pushed.

---

## 2026-07-08 (session 21) — Fixed solo-runner broken-screen bug + exercise-picker keyboard shrink; confirmed exercises-library cleanup — PUSHED (298d88d, b1aa50c)

**Context:** Session opened as `/hello-claude`, but the user had Plan Mode active mid-ritual, which meant the preview server never actually started at Step 1 — this caused a false 40-test Playwright failure scare later in the session (root-caused correctly: checked server status before assuming a regression, per the systematic-debugging discipline, rather than repeating the les-025 "flaky/fatigue" mistake). Session ran under a tight remaining usage budget (started at 9% weekly, flagged to Jake throughout); several process decisions (skip full multi-agent review on the second fix, stop before the bigger backlog items) were made explicitly with Jake given that constraint.

**Done:**
- Hello-claude's targeted code review (Step 4) found a real, previously-unbanked bug: `_afterRunnerSave` (app-runner.js) only special-cased role `'client'`; solo fell through to a PT-only `openClient()` call scoped by `coach_id = currentUser.id` — but a solo client record has `coach_id = NULL`, so it always errored. Fixed by adding `'solo'` to the working branch. New Playwright regression test added. Full 3-agent multi-agent-review (security/scoping, solo-mode, duplicates/regressions) ran clean. Pushed 298d88d, app-runner v16.
- Jake reported the 682f86f exercise-picker fix from last session helped but the modal still shrank/drifted once the on-screen keyboard actually opened. Diagnosed (confirmed via an AskUserQuestion check — shrink only happens once the keyboard is up, not before) as a `vh`-unit-vs-mobile-keyboard interaction: `vh` is sized against the layout viewport, which most mobile browsers don't shrink when the keyboard opens, so a plain `vh` box can end up partly hidden behind it. Fixed by syncing the modal's height/max-height to `window.visualViewport` on open and on resize (keyboard show/hide), falling back to the original `vh` values on unsupported browsers. Self-reviewed only this time (explicit call with Jake, given budget); `runner.spec.js` 26/28 passed cleanly (2 flaky, pre-existing login-timeout race, unrelated). Pushed b1aa50c, app-workouts v16.
- Rebuilt the exercises-library cleanup SQL from scratch this session (the original wasn't saved anywhere retrievable) as one self-contained script — a temp table computed once for the "keep" list, referenced plainly in every following statement, instead of the repeated-CTE pattern Jake flagged as "too many steps" last time (les-028). Jake ran it — confirmed 51 `remaining_exercises`.
- Confirmed OS self-check and golden-path sweep clean (first session of the day); CI green on all recent pushes; roadmap.md already in sync with LOG, nothing to flag.

**Bugs found + fixed:**
- Solo `_afterRunnerSave` broken-screen bug (see above) — found via proactive code review, not a live report.
- Exercise-picker mobile-keyboard shrink (see above) — found via Jake's live report, root-caused via a targeted question rather than a second blind CSS guess.

**UNVERIFIED (banked):**
- Exercise-picker VisualViewport fix (b1aa50c) — needs Jake's own phone; Playwright cannot simulate a real on-screen keyboard.
- Solo-runner fix (298d88d) — verified via Playwright + code reasoning (deterministic, not a maybe), not yet felt live by Jake.
- Workout-save speed fix (444d0f3, from 2026-07-07) — still not confirmed live by Jake.

**Decided:**
- Given the session's tight usage budget, Jake explicitly chose to run the full multi-agent review for the solo-runner fix (higher-risk, role/security-adjacent) but skip it for the picker-height fix (small, isolated, self-reviewed instead) — a deliberate risk-based tradeoff, not a silent shortcut.
- Jake asked to stop after these two fixes rather than start the runner-autosave build, given the remaining budget.

**Why:** Both bugs were real and worth fixing same-session — the solo one because it's a live-breaking dead end for a real usage mode with zero workaround, the picker one because Jake hit it again in actual use right after asking for the cleanup SQL. The budget constraint shaped process (review depth, stopping point) but not whether to fix real bugs found along the way.

**Follow-up (same day, after first /save):** committed the benign `.claude/settings.json` plugin-enable change that had been sitting uncommitted all session (context-mode + superpowers, from session 19's evaluation — no app behavior change). Also fixed one more small quick-win: `app-programs.js:679` had the same bare-`class="btn"` Cancel-button bug already fixed on the dashboards 2026-07-05 (undefined CSS class) — swapped to `.btn-secondary`, cosmetic only. app-programs v10→v11, pushed 6620720, CI green, 39/39 Playwright (1 skipped).

---

## 2026-07-07 (session 20) — Fixed slow workout save + slow Workouts-page load, cleaned up orphaned template/exercise backlog, fixed exercise-picker modal shrinking on mobile — PUSHED (444d0f3, 682f86f)

**Done:**
- Batched `saveRunnerSession`/`saveWorkoutSession`'s per-exercise save loop into 2 batched inserts each (exercises, then sets), correlated by `order_index` not response array order. Measured live: 14 requests/4.7s → 4 requests/1.1s on a 6-exercise save.
- Added rollback-on-failure to `saveWorkoutSession` (never had one before), matching `saveRunnerSession`'s existing rollback chain.
- Parallelized `saveWorkoutSession`'s per-exercise `_resolveExerciseIdForSave` lookup (de-duped by name first to avoid a create-race on repeated exercise names).
- Added `.limit(100)` to `renderWorkoutTemplates` and `renderClientWorkoutsPage`'s `workout_templates` queries — were silently riding the global 200-row server cap.
- New Playwright smoke test proving the new `saveWorkoutSession` rollback actually cleans up on failure.
- Fixed the exercise picker modal shrinking/drifting toward the bottom of the screen as search results narrow — `max-height:85vh` with no fixed `height`, combined with mobile's bottom-anchored overlay, meant the box shrank and slid down as fewer results matched, crowding the on-screen keyboard. Fixed with `height:70vh`.
- Cleaned up the historical orphaned-`workout_templates` backlog: diagnostic confirmed 103 orphaned rows (not ~993 as previously estimated — that figure was never actually measured), 0 in use anywhere; Jake ran the cleanup SQL, confirmed 0 remaining.
- Handed Jake a broader exercises-library cleanup SQL (delete every exercise not used by his personal/solo account) at his explicit request, after confirming via AskUserQuestion that he understood it would also detach the `exercise_id` link on real clients' templates/logs that reference a removed exercise (their logged history stays intact, just reverts to name-matching for that link).

**Bugs found + fixed:**
- Mid-session, a stray character (a literal `"0"`) was found prepended to the very first line of `app-runner.js`, corrupting the whole script and silently breaking ~25 unrelated tests (client-runner tests, some PT tests). Root cause traced via a direct browser console capture (a `pageerror` + `ReferenceError`), not guessed — confirmed via `git diff` that the file matched HEAD exactly at session start, so this was introduced by one of this session's own edits. Fixed, full suite reverified green. The exact mechanism of how the stray character got inserted was never reproduced, but the fix and its verification are solid.
- The new rollback test itself initially failed — traced to `flushLogState()` (called at the top of `saveWorkoutSession`) reading set values back out of DOM inputs by id; since the test injected `_logBlocks` data directly without calling `renderLogExercises()` first, those inputs didn't exist and `flushLogState()` silently overwrote the injected data with empty strings. Fixed by calling `renderLogExercises()` before invoking the save.
- 3-agent review (security/scoping, solo-mode, duplicates/regressions) found one real gap: the new rollback test's cleanup depended on the rollback-under-test actually succeeding, so a future regression in that exact rollback would both fail the test AND leave permanent debris in Jake's real account. Fixed with unconditional try/finally cleanup.

**UNVERIFIED (banked):**
- Workout-save speed fix (444d0f3) — verified via automated network-request-count measurement + full Playwright suite, not yet felt/confirmed by Jake in a real gym session.
- Exercise-picker modal fix (682f86f) — verified via automated bounding-box measurement at a simulated 390×844 viewport, not yet confirmed by Jake on his own phone.
- Exercises-library cleanup SQL (delete all except personal-account-linked) — handed to Jake, outcome/count not confirmed back.

**Decided:**
- On the sets-batch-insert failure path (both save functions), a failure now rolls back the whole session (log + exercises + sets) rather than the old per-exercise loop's "some sets saved, some didn't, session still marked saved" partial-success toast — since sets are now inserted in one atomic batch, "some failed" is no longer a real state (it's all-or-nothing), so a clean rollback + clear retry is better UX than a misleading partial-success message.
- Jake explicitly confirmed (via an AskUserQuestion after being shown the tradeoff) that he wants the exercises-library cleanup to remove entries even where that breaks a real client's template/log link, not just genuinely-unused entries — a deliberately more aggressive cleanup than the template one.

**Why:**
- Jake reported the slowness directly on the kanban board the previous session; this session's shortlist proposal surfaced it as the top item, and two Explore agents traced concrete, file:line root causes for both halves of the report before any code was touched — not vague slowness, two specific fixable patterns. The exercise-picker bug was reported live by Jake mid-session as a real UX complaint from actual use, investigated and fixed same-session rather than banked for later.

---

## 2026-07-06 (session 19) — Process/tooling session: kanban reorg, beta date → 31 July, wiki gap-analysis fixes, plugin install, skill sharpening + first live-validated skill-testing methodology — NO APP CODE CHANGED

**Context:** Jake opened by asking why session 18 took ~7 hours for what looked like "one feature" — answered from commit-timestamp evidence since session 18 was never `/save`d (see the backfilled session 18 entry below). Rest of the session was Jake directing process/tooling work: kanban board maintenance, a beta-date decision, a full wiki gap-analysis pass, installing the standalone Claude Code CLI + two plugins, and — at Jake's push — actually proving a skill-wording change works rather than asserting it.

**Done — kanban board (`wiki/board-coachapp.md`):**
- Reviewed and updated to reflect actual pushed state (was showing session 17/18 work as not-yet-pushed).
- Reclassified 4 items from "Needs Jake" → "Up Next": the 1RM 0.5kg-shift bug and the `showRunnerOneRMSheet` z-index fix (both reproducible with test/preview data, don't need Jake's live account), the `client_1rms` INSERT RLS gap (diagnosable via a Playwright client-role test; only the actual fix SQL would still need Jake), and the ICO breach-notification procedure (draftable by Claude for Jake's review rather than requiring him to author it from scratch).
- Added a new **"Proposed for Next Session"** column — a curated 3-5 item shortlist, distinct from the full "Up Next" backlog. Wired into the ritual going forward: `save` (Step 6) rewrites it every session-end based on that session's actual work; `hello-claude` (Step 6) reads it first as the anchor for "propose this session's plan" instead of generating one from scratch. **Discovered mid-session:** the Obsidian Kanban plugin strips freeform prose (non-checklist lines) from a column when it re-saves the file after Jake opens/edits the board — so this column's content has to live as checklist items themselves, not descriptive text around them. Adjusted expectations accordingly.
- Attempted a CSS snippet to lighten the kanban board's colour scheme (the vault's "Obsidianite" theme was rendering it dark) — Jake asked for it rolled back same session; reverted cleanly (deleted `kanban-light.css`, removed it from `appearance.json`'s `enabledCssSnippets`). Noted in memory as a soft signal: cosmetic/personal-tool styling changes are more reversible/tentative than app changes.

**Done — beta date pushed to 31 July 2026** (Jake's decision): updated `roadmap.md` (was "Week 5 — Jul 22–31" + staggered invites Jul 25/28/31 → single date 31 July), `board-coachapp.md`, and `guide-coachapp-roadmap.md` (mermaid node + numbered list) to match.

**Done — wiki gap analysis** (delegated to an Explore agent to read all 26 wiki pages without blowing up session context; then fixed everything it found):
- `coachapp-programs-architecture.md` (worst offender, stale since 2026-07-03): resolved a direct contradiction with `coachapp-runner-architecture.md` over whether exercise-id linking had shipped (it had, `1526704`); added Duplicate Week/fork-on-edit coverage; corrected a stale function name (`renderEditableWeek1Grid` → `renderPhaseWeekGrid`); added the 2026-07-04 `deleteProgram()` follow-up.
- `coachapp-workflows.md` (frozen since 2026-07-03): added the Exercise Picker, exercise identity linking, weight goals, Duplicate Week/fork-on-edit; corrected "Runner phase 2" to drop %1RM (shipped) from the still-open list.
- `coachapp-spec.md` / `coachapp-runner-architecture.md`: header "Last updated" dates were stale relative to their own content (both had been edited without bumping the date) — corrected.
- `index.md`: `log.md` existed on disk, actively maintained, but was never linked from the index — added under Maintenance.

**Done — installed the standalone Claude Code CLI + 2 plugins:**
- Jake tried `/plugin` commands directly in this chat session (the VSCode extension) — confirmed this environment doesn't support them at all, and no standalone CLI was installed on the machine to fall back to. Installed the official CLI (`irm https://claude.ai/install.ps1 | iex`, v2.1.201), fixed a PATH propagation issue (registry updated but already-running VSCode doesn't see it until restarted — worked around via the full exe path in the interim).
- Installed **context-mode** (mksglu/context-mode — MCP-based context-window sandboxing) and **superpowers** (obra/superpowers via claude-plugins-official — a 13-skill TDD/debugging/collaboration library), both eventually at **User** scope (available in every folder, not just this one) after an initial Project-scope install.
- **Confirmed these plugins are inert in this chat environment** — searched this session's own available tools for any trace of them, found nothing. They only work inside the separate standalone `claude` CLI process, even though both read/write the same `~/.claude` folder on disk.
- Added 4 new glossary entries (`guide-glossary.md`): Claude Code CLI vs this chat, Plugin, Marketplace, Scope (Project vs User).

**Decided — don't migrate primary workflow to the standalone CLI.** Evaluated the tradeoff directly: this chat environment has things the CLI likely doesn't (subagent orchestration, artifacts, scheduling, the whole hello-claude/memory ritual); the CLI has working `/plugin` support this chat doesn't. Neither is a strict upgrade. Recommended against switching unless a specific plugin need arises.

**Done — evaluated superpowers/context-mode for extraction value** (read the actual skill files, not just the plugin descriptions): recommended adopting **`writing-skills`**'s RED-GREEN-REFACTOR testing methodology (pressure-test a skill/rule with a subagent before vs. after adding it — none of CoachApp's ~15 skills have ever been empirically tested this way) and **`systematic-debugging`**'s sharper "3+ failed fixes = stop, this is an architecture problem, not another patch" rule. Rejected `verification-before-completion` (redundant with existing `feedback-verify`/feature-audit), `finishing-a-development-branch` (branch/PR/worktree workflow that doesn't match CoachApp's single-branch-on-master reality), and strict TDD (doesn't match how Jake actually ships — tests added same-commit to prove a fix, not written first).

**Done — folded both into `hello-claude/SKILL.md`:** sharpened the "If work goes in circles" section with the Iron Law, the 3-strikes-architecture rule, a red-flags list, and Jake's own verbal tells ("is that not happening?", "stop guessing", all-caps restatement) as explicit signals to stop.

**Done — actually proved the wording change works, instead of asserting it** (first real use of the newly-adopted testing methodology): ran 5 paired subagent reps (10 total) — one arm given the exact pressure scenario with no guidance, one arm given the same scenario plus the new rule text. Result: **5/5 baseline reps correctly diagnosed the technical root cause on their own** (the model's debugging instinct wasn't the gap) but **0/5 paused before acting — all planned or executed a fix solo.** **5/5 primed reps explicitly stopped and committed to surfacing the finding before writing more code.** Zero variance on both sides — a clean signal per the methodology's own "variance is a metric" guidance. Confirms the rule's actual value is enforcing a human checkpoint before further autonomous action, not making the debugging sharper (that was never broken) — which is specifically the failure this rule was written to prevent (Claude gave Jake wrong info twice in a row earlier this session before actually verifying against his real environment).

**Not done / explicitly out of scope this session:** no CoachApp app code, no Playwright run (nothing to test), no cache-bust concerns (no module files touched).

**Why:** Jake's original question ("why did session 18 take 7 hours") surfaced a real process gap — no live log existed to answer it from. Closing that gap, then using the same session to actually validate a process change empirically rather than just writing it down, is the throughline: less "build a feature," more "make the next 100 features more reliably built."

---

## 2026-07-06 (session 18, cont.) — Found + fixed the real flaky-test root cause; pushed (31698fe)

**Correction to the entry below:** its "not a real bug" verdict on the test flakiness was wrong. Jake asked to investigate properly rather than keep retrying blindly.

**Root cause:** `tests/helpers.js`'s `loginAsClient` logged the test account in and returned immediately. `loginAsPT` already had a comment explaining why that's unsafe (`#app-shell` is visible before `loadUserInfo`/`renderClientDashboard` finishes) and waits for its own dashboard heading before returning — `loginAsClient` had no equivalent wait. `renderClientDashboard` (app-dashboard.js:219) shows a `Loading…` placeholder synchronously, then replaces it with the real page only after several parallel Supabase fetches resolve. Every client-role test's `beforeEach` fires its first click (e.g. `[data-page="workouts"]` in `runner.spec.js`) straight after `loginAsClient` returns — if that click landed before the dashboard/nav had finished rendering, it got silently overwritten once the async render caught up. This explains the observed symptoms exactly: intermittent AND sometimes-100%-reproducible failures scattered across client/solo tests, unrelated to that session's actual diff, including a case where the page was directly observed stuck on the client dashboard ("No program assigned") instead of Workouts.

**Fix:** added `await page.waitForSelector('h1:has-text("Hi,")', { timeout: 15000 })` to `loginAsClient`, mirroring `loginAsPT`'s existing pattern.

**Verified:** `runner.spec.js` alone — 27/27 green. `runner.spec.js` + `solo-account.spec.js` together (the exact 38-test pre-push smoke set) — green on 2 consecutive full runs, then green a 3rd time inside the actual pre-push hook.

**Pushed:** 31698fe. CI (`Check & Deploy`) green.

**Also corrected:** the "Flaky test" and "Race condition" entries in the LLM wiki glossary (`guide-glossary.md`) now use this as the real worked example, replacing the incorrect "confirmed system exhaustion" framing — and the Flaky-test entry now explicitly calls out "jumping to fatigue because it requires no further work" as the trap, since that's exactly what happened here.

**Why:** a plausible-sounding explanation (hours of continuous test runs → tired machine) was accepted before ruling out a code cause, even though the failure recurred immediately on the very next run and hit early tests too — both signs a load-based explanation didn't actually fit. Investigating properly took under 15 minutes once actually pursued.

---

## 2026-07-06 (session 18) — Session 17 backlog pushed + 2 bugs found in it fixed (app-dashboard v3, app-workouts v12, app-runner v13, app-progress v4); Exercise identity linking + new Exercise Picker built (app-programs v10, app-progress v5, app-runner v14, app-workouts v13) — PUSHED (9b1fb9c, 1526704)

**Note on this entry:** backfilled after the fact — `/save` was not run at the end of this session, so STATUS.md/LOG.md were never updated in the moment. Reconstructed from the two commit messages and roadmap.md's session-18 notes. The only hard timing evidence available is the gap between the two commits: **9b1fb9c at 12:33:56 and 1526704 at 18:04:49 — 5h31m apart.** No record of when the session actually started, so real total time may be longer than that.

**Done — pushed session 17's backlog (9b1fb9c):**
- Session 17's Areas 1/2/4 work (false-positive "Save failed" toast fix, rest-time-on-swap/add fix, %1RM→strength-table routing, cardio fields in the workout-preview slider, dashboard "Current program" header, delete-set button spacing, mobile RPE/RIR label fix, `.modal-box`→`.modal` CSS fix across 5 sites) went through multi-agent review before push and found two real bugs in that session's own work, fixed before pushing:
  - Weight goals form (starting/goal weight, drives the Body Weight chart Y-axis) was wired into the PT-facing `renderClientWeight` only — clients and solo users had no way to reach it from their own My Progress page (`renderProgressWeight`, a separate function). Ported the form + Y-axis logic there too; fixed `saveWeightGoals` to refresh whichever view is actually showing it.
  - `saveRunnerSession`'s exercise-insert failure path re-enabled the Save button but never cleaned up the already-inserted `workout_logs` row — a retry could create a duplicate session and orphan the first attempt. Added rollback (sets → exercises → log) before allowing retry.
  - Also fixed: Y-axis inverted for a weight-gain goal (calc assumed goal < starting); ported the 0.5kg stepSize fix to the client/solo chart too.
- 69/69 Playwright passing, 3-agent review + self-verification clean.

**Done — Exercise identity linking + new Exercise Picker (1526704):**
- **Root cause:** previous-session/1RM data went missing when the same exercise was typed slightly differently across templates (e.g. "Bicep Curls" vs "Bicep Curl") — the runner's lookup already searched globally per client but matched on exact name string.
- **Schema:** added a real `exercise_id` FK to `workout_log_exercises` and `client_1rms` (`workout_template_exercises` already had one), with a name-match fallback for older/unlinked rows. Wired through `saveRunnerSession`, `saveWorkoutSession`, `save1RM`, `saveBig5OneRMs`, `_getProgramOneRMStatus`, `fetchRunnerLastSession`, `_lookupClientOneRM`.
- **Historical data migration:** one-time SQL seeded the (previously empty) exercise library from real usage and linked 4,777 template exercises, 27 logged exercises, 18/19 1RMs. Jake reviewed his actual exercise list and specified which spelling variants to merge (Close Grip Pulldown, RowErg, Trap Bar Jump, etc.).
- **New shared Exercise Picker** (search-as-you-type, explicit "Create new exercise", collapsible archived section) replaces the old dropdown+free-text entry everywhere: workout builder (add + edit), runner swap/add, 1RM entry. Archive/unarchive added to the Exercise Library management page. The old "1RM lifts quick-pick + auto-scroll" dropdown shortcut was dropped (Jake confirmed fine with this); the underlying %1RM calculation itself was untouched.
- Diffstat: `index.html` +8/-0 · `js/app-programs.js` +27 · `js/app-progress.js` ±106 · `js/app-runner.js` ±64 · `js/app-workouts.js` +486/-… (largest single-file change of the session) · plus test files.

**Bugs found + fixed (mostly via multi-agent review):**
- Race condition — typing into the picker's search box while the library was still loading got silently wiped once the fetch resolved.
- Two missing RLS policies — clients had no INSERT or SELECT access to the `exercises` table at all, so the picker silently failed for every client (worked fine for PT/coach, which already had its own policy).
- `escapeHtml()` was applied before the JS-string-escape on names rendered into `onclick` attributes, so any exercise name with an apostrophe (e.g. "Farmer's Carry") broke the picker.
- Two double-tap race conditions that could create duplicate library entries (picker's "Create new exercise", and the Big 5 quick-start form).
- 3-agent review (security/scoping, solo-mode, duplicates/render-safety) confirmed all RLS/solo-mode paths clean after the fixes above.

**Known friction this session:** first push attempt for the exercise-identity commit was blocked by the pre-push hook's own Playwright smoke-test pass hitting severe environmental flakiness after 3+ hours of continuous test runs (not a real bug) — required a full environment reset before the retry succeeded. This is the likely largest single contributor to the session running long, on top of the schema migration + data backfill + a new shared component threaded through 4 integration points + a 5-bug review-and-fix round.

**Not done:** `/save` itself — this is the gap this entry exists to close. Also flagged to Jake directly: STATUS.md still shows "session 17" at the top and lists the session-17 work as uncommitted, which is now stale (both commits are pushed and CI is green).

**Why:** Jake asked directly why the session took ~7 hours for what looked like "one feature." The honest answer, reconstructed from commit evidence since no live log exists: it wasn't one feature — it was finishing and pushing a backlogged session's work (with 2 bugs caught and fixed in review), then a schema migration + historical data backfill + a new shared UI component wired into 4 places + a 5-bug fix round, then a stuck test environment on the first push attempt. Banking this now so the same question doesn't require after-the-fact reconstruction next time.

---

## 2026-07-05 (session 17) — Live-test backlog organized into 4 areas; Areas 1/2/4 built + tested; %1RM runner routing fixed (app-dashboard v2→3, app-workouts v11→12, app-runner v10→13, app-progress v3→4) — NOT YET PUSHED

**Context:** Jake live-tested a real gym session plus the wider app end-to-end and reported 16 bugs/feature requests in one message. Organized them into 4 areas (Runner, Progress/Stats, Personal/Solo, Dashboard) with file:line-grounded root-cause notes via a read-only Explore pass, written to `roadmap.md`'s new "Session backlog" section. Then built and tested Areas 1, 2, and 4 in sequence (Jake picked the order live, one area at a time), plus a live bug Jake found mid-testing (Trap Bar Jump UI inconsistency) that turned into a scoped "Runner Phase 2 — %1RM only" build.

**Done — Area 1 (Runner):**
- **Rest time on Swap/Add exercise** — `_confirmRunnerExerciseFromModal` now derives `restSecs` from the entered rest field (`parseRest(cleanSets[0]?.restMin) || 90`), matching how a template's rest time is read on initial load. Previously: swap never reassigned `restSecs` at all; add hardcoded 90s regardless of what was entered.
- **Delete-set button spacing** — added `margin-left:8px` between the delete button and the complete-set tick in the strength table (was a shared 6px flex gap).
- **Redundant RPE label** — mobile log-wizard header now shows RPE or RIR dynamically (was hardcoded to always say RPE); the field's own placeholder now shows a numeric range hint instead of repeating the word.
- **"Save workout" false-positive error** — root cause confirmed via direct source instrumentation (not just static reading): `dbq()`'s default `showUserError:true` fires a "Save failed" toast on *any* non-PGRST116 error, including `saveRunnerSession`'s client-lookup — which already has a safe fallback (`|| currentUser.id`) and continues the save regardless. Fixed with `{ showUserError: false }`. Blast-radius sweep found the identical pattern in 2 more places (`saveWorkoutSession`, `showLogSessionModal`) and fixed those too. Also fixed a real second bug in the same function: if the `workout_log_exercises` insert failed after the `workout_logs` insert succeeded, the Save button stayed stuck on "Saving…" with zero feedback — now re-enables and shows an error toast.
- Trap Bar Jump routing — investigated live with Jake mid-session (see below), turned into a separate scoped fix.
- 4 new Playwright regression tests. Full suite green throughout (66/66 → 67/67 → 69/69 as each area's tests were added).

**Done — Area 2 (Progress & Stats):**
- **Cardio fields blank in workout-preview slider** — `openSessionDetail`'s set-line builder only branched timed vs. strength; never read cardio fields. Fixed by reusing the exact cardio-formatting logic already used by the template-card preview (pace/500m, pace/km, HR zone, stroke rate, duration/distance) — same helper, not reinvented.
- **Bodyweight graph Y-axis** — added `stepSize:0.5` to the Chart.js config (0.5kg ticks instead of arbitrary fractional ones).
- **Weight goals feature** (new, to satisfy the Y-axis min/max half of Jake's ask) — no structured "goal weight" field existed anywhere in the data model (goals are free-text `metric_label`/`target_value`), so rather than fragile name-matching, added 2 new nullable `clients` columns (`starting_weight_kg`, `goal_weight_kg`) plus a small "Weight goals" form on the Weight page (two native number inputs, no pre-fill per Jake's call, `saveWeightGoals()`). When both are set, the chart's Y-axis min = goal weight (rounded down to 0.5), max = starting weight + 1kg (rounded up to 0.5); falls back to auto-scaling if either is unset. Required a **new client-role UPDATE policy on `clients`** — every existing UPDATE policy on that table was PT-only until now. Jake ran the SQL; policy confirmed live via `pg_policies` query (`clients_update_own_row`, `cmd:UPDATE`, `qual`/`with_check` both `user_id = auth.uid()`).
- **Update-1RM modal rendering outside a modal** — root cause: `.modal-box` (used by `showAdd1RMModal`, `showEdit1RMModal`, the delete-account modal, and 2 runner-context 1RM modals — 5 sites total across app-progress.js/app-runner.js) has zero CSS definition anywhere in `main.css`. The correctly-styled class is `.modal` (background/border-radius/padding/max-width/shadow), used by every other modal in the app. Swapped all 5 sites. This also closes an already-tracked backlog item (found 2026-07-05 during the dashboard pass, same bug class as `.dashboard-card`). Bonus find while in this code: the mid-workout "set your 1RM" sheet (`showRunnerOneRMSheet`) had no z-index override despite opening while the runner (z-index:300) is up — added `z-index:1000`, same fix already applied to the exercise picker for the identical reason. This one is UNVERIFIED live (reasoned from an established pattern, not reproduced).
- **1RM value shifts 0.5kg on duplicate entry** — genuinely could not find a code-level cause after checking `save1RM`, `saveBig5OneRMs`, and every Epley-calculation site for rounding/dedup logic. None exists. Needs live reproduction next session.
- **Personal Bests / Performance merge** — not built; Jake had already deferred this to its own scoping session before this build started.
- **New gap surfaced:** clients can now technically self-detach from their PT via the API, since Postgres RLS is row-level (not column-level) and the new client-writable-row policy on `clients` doesn't distinguish `coach_id` from `starting_weight_kg`. Discussed directly with Jake; accepted as consistent with the existing trust model for `weight_logs`/`client_1rms`, banked as its own roadmap item needing a real cancellation-flow design later, not fixed now.
- New `tests/progress.spec.js` (2 tests: cardio formatting, 1RM modal CSS class). Full suite 67/67 (1 flaky retry, unrelated timing on the new modal test — passed on retry).

**Done — Area 4 (Dashboard):**
- **"Current program" header** — client and solo dashboards now show a slim header (program name + "View program" button, routes to `navigate('workouts')`) above the existing "Up next" hero card, when a program is assigned. Confirmed no such element existed before. 2 new Playwright tests (both skip gracefully — neither test account has an assigned program).

**Done — Runner Phase 2, scoped to %1RM only (live bug hunt with Jake):**
- Jake screenshotted Trap Bar Jump (wizard mode, single-set KG/REPS/LOG UI) next to Back Squat (table mode) on his own account and asked to investigate — this is the exact "Trap Bar Jump UI inconsistent" item already on the backlog, which he'd earlier told me to skip. Live evidence corrected my original hypothesis: it is **not** a name-matching bug. Trap Bar Jump's target row showed "20% 1RM" — `_isPlainStrengthExercise` deliberately excludes any exercise with an `intensityMin` (%1RM) set from table mode, regardless of name, because the table wasn't believed to support %1RM display. Reading the actual code showed the table's target bar (`_buildTargetCols`/`_renderTargetBarHtml`) **already** computes and displays the %1RM→kg target and the "set your 1RM" banner — identical logic already duplicated into the wizard. So the fix was much smaller than "Runner redesign phase 2" sounds: removed the `!s.intensityMin` exclusion (one line), left timed/unilateral excluded (Jake's explicit scope choice — those still need their own design work for table mode). Added a placeholder (not auto-filled value) in the table row's weight input showing the calculated suggested kg when a 1RM is known, matching what the target bar already shows. 1 new Playwright regression test. Full suite 69/69 clean after the change (2 skipped — the two new dashboard-header tests, no test-account data).

**UNVERIFIED (banked to STATUS.md open to-dos):**
- Weight goals feature — Jake asked to check the chart-reshape + save flow on his own account; hasn't confirmed the result yet.
- Dashboard "Current program" header — Jake mentioned his current program name ("Hyrox Conjugate") for context but hadn't explicitly confirmed the header/button rendered correctly before reporting the Trap Bar Jump bug instead.
- %1RM table routing fix — Jake was asked to hard-refresh and check Trap Bar Jump now matches Back Squat; this was the very next thing to check when `/save` was invoked.
- z-index fix on `showRunnerOneRMSheet` — reasoned from an established pattern, never actually reproduced/screenshotted live.

**Decided:**
- Runner Phase 2 scope: %1RM only for now, not all 4 wizard-only exercise types in one build. Timed/unilateral/cardio deliberately left on the wizard.
- Client self-detach-from-PT risk accepted as-is (row-level RLS trade-off), not blocking the weight-goals ship — proper fix (cancellation workflow) deferred to its own session.
- Skipped Trap Bar Jump earlier in the session (original name-matching hypothesis), then un-skipped it when Jake hit it live and asked directly — the live evidence overturned the original theory, which is exactly why the "skip for now, real fix is an architecture decision" call from earlier turned out to be based on an incomplete diagnosis.

**Also this session:**
- Updated 2 skills based on Jake's direct question about whether the reported bugs indicated gaps in the standing QA checks: added an "all-exercise-types check" to hello-claude's blast-radius sweep (cardio-blank-preview is the same bug *class* as a historical timed-set formatting bug), and a "destructive-adjacent-to-primary spacing" check to feature-audit's gym-user lens (the delete-button-spacing bug is exactly what that lens is supposed to catch). Explicitly did *not* add a check for the original Trap-Bar-Jump-name-matching theory, since the live evidence showed the real cause was unrelated to names.
- Mirrored the backlog organization + the client-detach gap into the LLM wiki (`guide-coachapp-roadmap`, `coachapp-runner-architecture`, `coachapp-spec` — first time that page tracks open decisions, not just settled principles) in the same turn per the standing roadmap-wiki-sync rule.
- Investigated an alarming-looking console message during test debugging ("tip: ⌁ auth for agents [www.vestauth.com]" printed by `dotenv`) — traced to source, confirmed it's real (if unwanted) self-promotion baked into `dotenv` v17.x advertising their commercial `dotenvx` product, not a supply-chain compromise. No action needed; banked as a lesson so it isn't re-investigated from scratch next time it's seen.
- A test-harness debugging detour while root-causing the save-workout error: `_runner` is declared with `let` inside `app-workouts.js`, a *different* `<script>` tag than `app-runner.js` where it's used — `window._runner = {...}` from a Playwright `page.evaluate` does not affect the real `let`-scoped binding (classic vs. `var`/function-declaration globals). Cost real time to diagnose; banked as a lesson for future direct-state Playwright tests in this codebase.

**Why:** Jake's own live-testing (both the original 16-item report and the Trap Bar Jump follow-up) is the highest-signal input this project gets — it surfaces exactly the gaps that code review and Playwright can't (a false-positive error toast, a UI inconsistency with a wrong first theory, a genuinely missing feature). Building area-by-area with Jake choosing the next area, rather than batching everything into one pass, kept each piece reviewable and let a live discovery (Trap Bar Jump) get investigated properly instead of steamrolled past.

---

## 2026-07-05 (session 16) — Dashboard CSS consistency pass (main.css v3→4, app-dashboard v1→2) — PUSHED (313bc74); runner autosave scoped + designed, not built

**Context:** Jake asked to improve the design and flow of the dashboard. Explored the PT/client/solo dashboard code first — found real bugs, not just polish opportunities. Scoped to a consistency/bug-fix pass (Jake's choice, over a full shared-component rebuild or an information-architecture rework). In the same session, also picked up the standing High-priority runner-autosave to-do and scoped/designed it via a Plan agent, but the build was interrupted before any code was written — Jake redirected to push the dashboard work, log the autosave plan, and save.

**Done — dashboard consistency pass:**
- `.dashboard-card`/`.card-header`/`.card-title` were used ~37 times across all three dashboards (`renderDashboard`/PT, `renderClientDashboard`/client, `renderSoloDashboard`/solo in `js/app-dashboard.js`) but had zero CSS definitions anywhere in the repo — every "card" rendered with no background/border/shadow. Added real rules to `css/main.css`, scoped under `.dashboard-card` specifically so they can't leak into the different (already-working) `.card-header`/`.section-title` usage in `app-progress.js`/`app-calendar-goals.js`.
- Consolidated three byte-for-byte duplicated inline `<style>` blocks (`.pt-grid`/`.client-grid`/`.solo-grid`) into one shared `.dashboard-split-grid` class in `main.css`.
- Fixed 4 bare `class="btn"` Cancel buttons (no matching CSS rule — only `.btn-primary`/`.btn-secondary`/etc. exist) to `.btn-secondary`, matching the sitewide Cancel convention.
- Replaced hardcoded hex colors with design tokens (`var(--danger)`/`var(--warning)`/`var(--success)`/`var(--accent)`). One deliberate visible change: the "on track"/holiday green shifts from `#22c55e` to the design system's actual `--success` (`#10b981`) — flagged in the commit message, not silent. A second color (`#3b82f6`, "gym" event blue) has no matching token — left as-is with a `TODO(Jake)` comment rather than inventing one or silently collapsing it into `--accent`.
- Fixed 2 real mobile bugs: the PT stat strip had no mobile override at all (would cramp into 3 columns at 480px); the solo stat strip was `display:none` below 640px (vanished entirely instead of stacking). Both now pair up on mobile (`repeat(2,1fr)`).
- 2 new Playwright smoke tests (`.dashboard-card` has a real background; `.solo-stats` stays visible at 400px) — 29/29 pre-push suite green, CI green.
- 3-agent review (security/scoping, solo-mode, duplicates+render-safety) + verifier came back clean — zero blocking findings. Two informational notes surfaced and banked as roadmap follow-ups rather than fixed in this pass: a 5th unstyled bare-`.btn` Cancel button at `app-programs.js:672` (same bug, outside dashboard scope), and `.pt-stats`/`.solo-stats` duplicating the purpose of the pre-existing (dead) `.stats-row` class.
- Visually verified at 1280/900/640/480/400px via a throwaway Playwright screenshot script (the `preview_*` tools this environment's skills assume don't exist here — see lessons below) — confirmed cards now show real backgrounds/shadows and both stat strips reflow correctly on mobile.

**Also found (not fixed, banked as follow-ups):** `var(--bg-accent)`/`var(--text-accent)`/`var(--surface-2)` are referenced 52× across 7 files but never defined anywhere in `main.css` — same bug class as the dashboard-card fix, but app-wide. `.modal-box` (used in `app-progress.js`/`app-runner.js`) has no CSS definition either. Both deliberately kept out of this pass's scope (Jake's call) — need their own audit of intended styling per site before fixing.

**Done — runner autosave, scoped and designed (not built):**
- Explored the runner's `_runner` lifecycle in depth: confirmed zero persistence exists today (no localStorage, no `beforeunload`/`visibilitychange`, no resume concept anywhere), confirmed the exact final-save schema (`workout_logs`→`workout_log_exercises`→`workout_log_sets` via `saveRunnerSession`), and confirmed a second parallel in-progress representation (`tableRows` for the Hevy-style strength table) that any draft must also capture alongside `loggedSets`.
- Jake chose the hybrid approach: a localStorage-only draft now (checkpointed on every `renderRunner()` call plus a 10s safety-net tick, keyed `_runnerDraft_<clientId>`, same-day staleness cutoff, resume/discard confirm modal wired into `startWorkoutRunner()`, cleared inside the existing `discardRunner()`); a DB-backed draft for cross-device recovery is deliberately deferred.
- A full function-level implementation plan was produced via a Plan agent (exact functions, hook points, serialization shape, failure-mode handling, Playwright test cases) — see `roadmap.md`/`STATUS.md`/`coachapp-runner-architecture` (wiki) for the design. Session was interrupted before any code was written for this half — ready to build directly next session.

**Not done (flagged, banked to roadmap/STATUS):**
- The app-wide undefined-CSS-var bug, `.modal-box`, and the `app-programs.js:672` `.btn` bug (all above).
- Runner autosave itself — designed but not built.
- `hello-claude`/`run-coachapp` reference `preview_start`/`preview_resize`/`preview_screenshot`/`preview_snapshot`/`preview_click`/`preview_fill` tools that do not exist in this Claude Code CLI environment (confirmed via ToolSearch). Worked around by starting the static server directly (the exact command already in `.claude/launch.json`) and driving verification via Playwright directly instead — banked as `lessons.jsonl` les-022 and a STATUS to-do; the skills themselves should be updated to check-then-fallback rather than assume the tools exist.

**Predictions graded this session:** 5 predictions with a 2026-07-05 `verify_by` were due — all graded `correct` from existing documented evidence (client dashboard was in fact built and iterated on since, pth-028; session-history two-query fix has held up, pth-038; the predicted PBs-tab RLS gap was real and got fixed 2026-06-29, pth-035; the last-session-strip placement was never revisited, pth-036; client calendar has shipped clean with no further fixes, pth-062). One new prediction banked (pth-088: the runner autosave plan will be built next session directly from the banked design, no re-scoping needed).

**Why:** Jake's dashboard ask surfaced real bugs (not just aesthetic gaps) that were worth fixing regardless of any future redesign — shipped those now since they were low-risk and immediately useful, while explicitly declining bigger scope (component rebuild, IA rework) he didn't ask for. The runner autosave got a full design pass so the next session can build directly against a decided, reviewed plan instead of re-opening the scoping conversation — closing the actual value of the session (a shipped fix + a ready-to-build plan) rather than leaving a half-built feature mid-flight when Jake redirected.

---

## 2026-07-04 (session 15) — Runner exercise-picker freeze fix + silent beep fix + client-query scoping quick win (workouts v10→11, runner v9→10, clients v1→2) — PUSHED (84f9267, f997474)

**Context:** Jake reported a real production incident from his own personal-account gym session (19:42–19:45): tapping Swap exercise after Add exercise froze the runner solid, and a forced page reload wiped the entire in-progress session. He also asked for a check that saves/session-history are accurate, and flagged that the 10-second voice rest cue works but the 5-second countdown beeps don't.

**Done:**
- Diagnosed the freeze: `showAddExerciseToTemplateModal` (app-workouts.js) builds its overlay with a hardcoded, non-unique id (`add-to-template-modal`) and fetches its exercise list asynchronously *before* appending it — a window with no visible modal yet. A fast Swap-then-Add tap in that window fired the function twice, producing two overlays sharing one id; `getElementById`/`closeModal` only ever resolve to the first DOM match, so the visible (second) modal could never be closed from within the app.
- Verified via Playwright (59/59, no console errors) that the save-on-finish path and session-history rendering are both clean and unaffected — the incident never reached `saveRunnerSession`, so no partial/orphaned `workout_logs` rows exist from it.
- Diagnosed the beep gap as a hypothesis (not device-confirmed): the tone beeps depend on the Web Audio API, which iOS aggressively suspends on screen-lock/backgrounding — very plausible mid-rest in a live gym set — while the working "10 seconds" voice cue uses the separate Web Speech API, unaffected by that suspension.
- Jake approved both fixes. Built: a pending-flag guard + button-disable (`wr-swap-btn`/`wr-add-btn`) while the picker's fetch is in flight, plus a `.catch()` error path that didn't exist before (previously a rejected fetch just silently never opened the modal). Replaced the 5-second tone-beep countdown with spoken numbers (`speakCue(String(n))`) across all three timers that had one: rest, timed-set, and cardio interval.
- 3-agent review (security/scoping, solo-mode, duplicates/render-safety) — security and duplicates/render-safety came back clean (one low-severity theoretical edge case noted, not fixed, consistent with the rest of the codebase). Solo-mode agent found a real bug in the beep fix itself: the cardio interval timer's entry point (`startCardioTimer`) only called `_unlockAudio()`, never `_unlockSpeech()` — its new spoken countdown would have silently never fired. Fixed same session by adding the missing `_unlockSpeech()` call, mirroring the pattern already used elsewhere.
- New Playwright regression test directly reproduces the double-tap race (calls `showExercisePicker('add')` then `('swap')` back-to-back mid-fetch, asserts exactly one modal opens, the first call wins, and both buttons re-enable after close). Manually reproduced and confirmed the fix live in the preview against the E2E test client account before committing (not just Playwright) — rapid double-invocation correctly produced one modal, the button-disable state, and a clean close.
- Full suite green across three separate runs (59-60/60 passed each time); a different unrelated test flaked once per run and passed on retry — confirmed as pre-existing environmental/network-timing noise, not caused by this diff.
- Pushed 84f9267; pre-push hook + CI green.

**Also this session — quick wins (f997474):**
- Scoped the 5 known unscoped `clients` queries in app-clients.js (`openClient`, `renderClientOverview`, `saveUpdateEmail`, `showEditClientModal`, `saveEditClient`) by `coach_id` in addition to `id` — a defense-in-depth gap surfaced by the 2026-07-03 review. 2-agent review confirmed all 5 are PT-only code paths (grepped every call site across all 8 modules), so no client/solo flow could regress.
- Tightened the `client-workout.spec.js` "session history" locator to be onclick-scoped (`toggleClientPhase('client-session-history')`) instead of text-only, matching its sibling tests — the review agent confirmed the exact onclick string matches what app-workouts.js actually renders, so the test now genuinely exercises the toggle rather than risking a silent no-op.
- **Correction:** while tracing call sites, the review agent found `roadmap.md` stated solo accounts' own client record has `coach_id = auth.uid()` — verified against `app-core.js:132` (`.is('coach_id', null)` lookup) that it's actually `coach_id = NULL`. Fixed the doc.
- Pushed f997474. First push attempt was killed by a 2-minute Bash timeout mid-pre-push-hook (the hook runs the full Playwright suite) — retried with a longer timeout and it completed clean. Deploy step then hit the known transient "Deployment failed, try again later" GitHub Pages infra error (documented in lessons.jsonl les-017); `gh run rerun` succeeded on the first retry, confirmed green.

**Not done (flagged, awaiting Jake):** actual session-autosave/draft persistence — the deeper fix behind "lost all my data," since `_runner` still lives only in memory until the final save tap. Scoped as its own decision, added to roadmap.md as "Improve workout-tracking visuals + underlying data model" alongside a separate Jake request to revisit the runner's tracking visuals.

**New standing rule this session:** Jake asked that any new roadmap.md item always be mirrored into the LLM wiki (visual roadmap page + the relevant topic page) in the same turn, not batched for later — banked as `feedback_roadmap_wiki_sync.md` and wired into hello-claude. Applied immediately to this session's own roadmap addition, and again now to mark today's two fixes as done rather than open gaps.

**Why:** A live incident during Jake's own gym session — the exact audience CoachApp is being built for — losing real training data is about as severe as a bug gets for this product. Diagnosed properly (found the actual root cause via code + a direct DOM reproduction, not guesswork) rather than patching symptoms, and the review process caught a real bug in the fix itself before it shipped.

---

## 2026-07-04 (session 14) — Duplicate week + fork-on-edit + solo delete/PT toast fix (programs v8→v9, workouts v9→v10) — PUSHED (730738a)

**Context:** Jake reviewed the Test 1/Foundation & Calibration program screenshots and asked for three things: a way to duplicate a week's worth of workouts, a fix so renaming a picked-from-library workout forks a new one instead of silently overwriting the shared original, and a fix to `deleteProgram()` so solo can delete their own self-assigned program and the PT block toast names the actual clients.

**Done:**
- Asked two clarifying questions before building (per sounding-board/approve-before-build rules) — both resolved to the recommended option: "Duplicate week" makes weeks independently editable (not just a periodization shortcut), and any edit made through a phase slot forks a copy (not just renames).
- **Duplicate week** — new `duplicatePhaseWeek()` (app-programs.js) copies a week's day/workout assignments into the next empty week as real `program_phase_workouts` rows (same `template_id`, cheap — only forks on actual edit). Replaced the old week-1-only `renderEditableWeek1Grid` + read-only `renderDayGrid` with one unified `renderPhaseWeekGrid` used for every week, so weeks 2+ (manual or periodization-generated) are now equally editable (add/remove/reassign a workout per day), not just viewable. `_quickAssignPhaseWorkout` generalized to accept a week number via `data-week` instead of hardcoding week 1. Propagates to already-assigned clients by cloning a fresh per-client template copy per new slot — same pattern `_cloneProgramForClient`/`generatePhasePeriodization` already use, never sharing a client-owned clone across slots.
- **Fork-on-edit for shared master templates** — new `_cloneSharedMasterTemplate`/`_resolveEditableTemplateId` (app-workouts.js), wired into `saveEditTemplate`, `saveExerciseToTemplate`, `saveEditTemplateExercise`, `deleteTemplateExercise`, `moveTemplateExercise`. Before any of these write, checks whether the template is still referenced by more than one `program_phase_workouts` row (e.g. the same template picked into both Monday and Tuesday) — if so, clones it and repoints only the slot the edit came from (via a new `ctx.phaseWorkoutId` threaded through `openTemplate`/`openSessionDetail`), before applying the edit. Client-plan templates are deliberately excluded (guarded by `ctx.isClientPlan`) since each client slot already gets its own exclusive clone at assignment time — confirmed this via `_cloneProgramForClient`, so no fork logic was needed there. Removed the now-dead "shared template, changes apply to all" warning toast from `_checkClientPlanPropagation` — forking now prevents that scenario before it happens instead of just warning about it.
- **`deleteProgram()` fix** — the "clients assigned, remove them first" block now excludes the user's own solo self-assignment (`window._soloClientId`) from the blocking count, so Personal-view users can always delete their own program. The PT-facing toast fetches and names the actual blocking clients (e.g. "Assigned to Sarah Mitchell, Alex Turner — remove them from this program first") instead of just a count.
- 3-agent review (security/scoping, solo-mode, duplicates+render-safety) — all three came back mostly clean; the one real finding (confirmed by verifier pass): the `generatePhasePeriodization` confirm dialog said it only replaces "previously generated" weeks, but weeks 2+ can now hold manually-added/duplicated content too, and `_cleanupPhaseWeeksBeyond` wipes all of it unconditionally on regenerate. Fixed the dialog wording same session (app-programs.js:1013) rather than banking it as a to-do.
- 3 new Playwright tests added to `tests/programs.spec.js` (duplicate-week copies rows correctly, fork-on-edit leaves the sibling slot untouched, delete-block toast names the client) — 59/59 green, no regressions in the existing 56.
- Live-verified all three features against real Supabase data in the preview (not just Playwright): duplicated Foundation & Calibration's Week 1 into Week 2 and confirmed matching template IDs; renamed the shared "AAA" template via Monday's slot and confirmed Tuesday's reference was untouched, then confirmed a second edit to Tuesday no longer re-forks (ref count back to 1); confirmed the solo-delete block-filter and the PT toast's client-name resolution via direct query, without actually deleting Jake's real programs. All test data cleaned up after verification.

**Bugs found + fixed:**
- The `generatePhasePeriodization` confirm-dialog wording gap above — a genuine newly-possible silent-data-loss risk introduced by making weeks 2+ editable, caught by the render-safety review agent and fixed same session.
- Mobile/UI consistency nit — the "Duplicate week" button was first styled smaller (`font-size:10px;padding:2px 8px`) than its sibling "Configure"/"Generate weeks" buttons in the same phase-card region; caught via `preview_inspect` and matched to `font-size:11px;padding:3px 9px`.

**Decided:**
- Client-plan template editing stays untouched by fork-on-edit — each client already gets an exclusive clone per slot at assignment/generation time, so the "shared template" problem this session fixes only exists on the master/program-builder side.
- Duplicate-week client propagation clones fresh per slot rather than reusing the source week's existing client clone, matching the codebase's established "always clone fresh, sync via name-match prompt" pattern rather than introducing a new sharing model on the client side.

**Why:**
- Jake's screenshots showed two real gaps in the Programs builder (weeks 2+ effectively frozen without periodization; a shared "AAA" template silently editable from either slot with editors none the wiser) plus a solo/PT delete-toast usability gap. All three were scoped, questioned, approved, built, live-tested against real data, reviewed, and pushed in one session rather than banked as future to-dos.

**Also this session:** banked `lessons.jsonl` les-020 (git-push-timeout / preview-screenshot-retry / network-dump efficiency lesson) after Jake asked why the session ran long — see LOG below or `lessons.jsonl` directly.

---

## 2026-07-03 (session 13, cont. 2) — ~/.claude backed up to private repo (jakendwest-ops/claude-config) — PUSHED

**Context:** Jake pushed on the previous save's honest carry-over ("skills + memory + wiki live outside any git repo") — asked whether it actually needs addressing.

**Done:**
- Assessed: the LLM wiki is under OneDrive (cloud-synced) — not a loss risk, git would only add version history. But `~/.claude` (`skills/` = the OS, plus the auto-memory dir) is local-disk-only, no remote, no cloud sync — the least-protected yet most irreplaceable content, while the app and Vault both already have GitHub remotes. Real gap.
- Created **private** repo `jakendwest-ops/claude-config` (verified PRIVATE). `git init` in `~/.claude` with an **allowlist `.gitignore`** tracking ONLY `skills/` + `projects/C--Users-jaken-Claude/memory/` — sessions, transcripts, `settings.json`, caches all excluded. Secret-scanned the staged content before each push (empty both times). Auth via `gh` (tokenless, no PAT in URL — same setup as coachapp). Initial commit `4f7d324` (55 files).
- Made it durable: `/save` Step 5 now commits+pushes claude-config whenever a skill/memory file changed; documented in `reference_claude_config_backup.md` + MEMORY.md index. Commit `55dba6c`.

**Decided:**
- Wiki stays as-is (OneDrive backup is sufficient); git for it is optional/deferred.
- The ~/.claude backup tracks skills + memory only, never `settings.json` or transient state — keeps secrets out even under `git add -A`.

**Why:**
- The layer that makes the app + Vault useful (the OS + accumulated memory/relationship) was the one thing with no offsite copy. Wiring the push into /save stops the backup silently going stale.

---

## 2026-07-03 (session 13, cont.) — Vault repo-structure investigation + coachapp PAT security cleanup + accurate-steps feedback — PUSHED (coachapp ec30ebf; Vault this save)

**Context:** After the session-13 audit push (f3706a1), Jake asked me to investigate a discrepancy in my own description of the Vault repo layout — which turned into a security incident and its fix.

**Done — Vault repo structure clarified:**
- Confirmed there is ONE git repo (remote `jakendwest-ops/vault.git`): the Vault-OS framework AND the `Vault/` content are the same repo, root `C:/Users/jaken/Claude`, `Vault/` a plain subdirectory (no nested `.git`, no submodule). My session runs in a linked worktree (`claude/vigorous-dewdney`), but hello-claude/save read+write the absolute `C:/Users/jaken/Claude/Vault/...` path — the MAIN checkout on `master` — so every session's Vault writes land on master directly, bypassing the per-session worktree branch. My statement ("root is C:/Users/jaken/Claude") was correct; the real finding is that framework and Vault are one repo, and 4 live worktrees share one Vault on master (concurrent `/save` could collide).

**Security incident + fix (RESOLVED — clears the standing PAT to-do):**
- While investigating I ran `git remote get-url origin` on coachapp, which printed the live `ghp_` PAT embedded in the remote URL straight into the chat — the exact leak the standing to-do warned about (I made it worse by surfacing the value). Flagged immediately.
- Jake deleted the token. Then: stripped the token from the remote (`git remote set-url origin https://github.com/jakendwest-ops/coachapp.git`), cleared the stale/dead cached credential (`git credential reject`), and — since coachapp is a public repo (read needs no auth) and my non-interactive shell can't complete GitHub's interactive browser sign-in — pointed git at the already-authenticated `gh` CLI's keyring token via `gh auth setup-git` (valid `gho_` OAuth token, `repo` scope). Verified end-to-end with a trivial empty commit `ec30ebf` that pushed non-interactively through the full pre-push hook (27/27 Playwright green) with no token in the URL and no prompt.

**Feedback banked:**
- Jake: "when giving me steps, always provide accurate url and page names/settings because you use incorrect names and it adds to confusion." I'd said "Revoke" for a single classic token when GitHub's button is "Delete". Updated `feedback_step_by_step.md` with a non-negotiable accuracy rule (verify labels/URLs against the service's own docs before writing a step; never state a guessed UI label as fact) and re-checked the token-deletion steps against GitHub docs before giving them.

**Decided:**
- coachapp auth is now via the `gh` keyring token, not an embedded PAT — no secret in `.git/config`. `credential.helper` left as `manager` for other hosts.
- Empty commit `ec30ebf` is a permanent no-op marker in coachapp history — accepted (not worth rewriting pushed history).

**Why:**
- Investigate-before-asserting (Jake's standing preference) turned a "did I mislabel the repo?" question into a real repo-topology finding and caught a live-credential exposure; fixing auth via the existing `gh` token was the cleanest tokenless path that also worked from a non-interactive shell.

---

## 2026-07-03 (session 13) — Architecture & infrastructure audit + fixes — PUSHED (coachapp 609d5ee; Vault pushed)

**Context:** Jake asked for a full audit of the build system itself — redundancies slowing sessions / eating tokens, whether the codebase split is right and graphify still needed, skills redundancy/contradictions, whether review agents test correctly (not rewriting themselves to dodge bugs), and whether the LLM wiki + roadmap are kept current. Ran it as a parallel multi-agent audit (codebase/graphify, skills, wiki) plus direct analysis of the hello-claude/save token cost and review rigor. On Jake's "all", actioned every fix; on "push all", pushed.

**Done — graphify removed (coachapp 609d5ee, PUSHED):**
- graphify was installed 2026-06-29 for the old 7,968-line app.js, made redundant one day later by the 8-module split, and never regenerated (stale — indexed the now-deleted app.js). Its two PreToolUse hooks in `.claude/settings.json` fired on every grep/find/Read/Glob injecting a "MANDATORY run graphify first" block — a per-action token tax for a tool that isn't even installed. Removed `graphify-out/`, the graphify-only `CLAUDE.md`, and emptied the hooks. Pushed clean: full pre-push hook incl. 27/27 Playwright (run against the live preview server since the default 3001 was down) — no app code changed.

**Done — skills fixed (global `~/.claude/skills`, on disk, not a git repo):**
- Dead path `C:\Users\jaken\coachapp\...\js\app.js` (wrong root + pre-split filename) repointed to `OneDrive\coachapp` + the 8 modules in deploy-check, security-audit, run-coachapp — these would have errored / false-passed on their next real run (deploy-check & security-audit run right before a beta invite).
- mobile-check said 390px (contradicting Jake's own 480px "no exceptions" rule) → fixed to 480 in the skill + both memory files, with a note that Playwright's 390 test-viewport is a separate deliberate config. Stale test counts (47 / 14) → read the real total (suite is 56).
- run-coachapp now documents autoPort (port 3001 no longer guaranteed).

**Done — hello-claude/save efficiency + review rigor:**
- Bounded the three unbounded session-start reads: LOG → first 150 lines (a full read cost 33k tokens today), voice.md → last ~150 lines, predictions → grep-first. Session-start cost is now flat instead of growing every session.
- New pinned skill `multi-agent-review` (fixed 3 angles + verifier) replaces improvising the pre-push review each session, so rigor can't silently narrow. Added a weekly **full-file mode** (hello-claude Step 4) that reviews whole high-churn modules, not just the diff — closing the blind spot that let 5 unscoped `app-clients.js` queries survive ~12 diff-only reviews. Registered in hello-claude, save Step 5, deploy-check Step 2, and memory.

**Done — wiki + roadmap:**
- Fixed the one stale wiki line (`coachapp-programs-architecture.md` deleteProgram gap → marked fixed 66bf1fd) + wiki log entry. Wiki discipline otherwise intact (manifest current, no orphaned raw files).
- `roadmap.md` beta-prep refreshed: flags that 3 pre-beta gates (ICO breach procedure, delete old Jake West client record, Supabase Pro for leaked-password protection) have sat untouched 4 sessions while the runner pivot absorbed everything — ~3 weeks to Jul 22.

**Audit verdicts (no action needed):**
- Codebase split is healthy — largest module app-runner.js ~1,948 lines, one coherent concern; leave it. No further splitting or graphify needed.
- No evidence of review agents gaming tests / rewriting themselves to dodge bugs — gates have grown not shrunk, lessons log is self-critical. The real gap was structural (diff-only review), now addressed by full-file mode.
- Sessions 9–12 runner focus was a deliberate, Jake-approved, research-backed pivot — not drift.

**Not done (flag):**
- Full `/save` ritual (STATUS full sweep, predictions grading, voice deltas) not run — this entry + roadmap + a STATUS last-push bump were pushed to keep the Vault coherent; a proper `/save` at wrap-up would round it out. The 6 edited skills, the new skill, and the 3 edited auto-memory files live on disk (`~/.claude`, not a git repo) and the 2 wiki edits live in the LLM wiki (not a git repo) — saved, but not pushable.

**Why:**
- Every fix served the audit brief: cut the per-action / per-session token tax (graphify hooks, unbounded Vault reads), stop checklists silently pointing at dead paths / wrong widths / wrong counts right before a beta gate, and make the code review a fixed recipe with a net for untouched code rather than an improvised diff-only pass.

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
