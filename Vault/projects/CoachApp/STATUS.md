# CoachApp — STATUS
_Last updated: 2026-08-12 (end of session) — **16 commits PUSHED and deployed
(`8e7573d..9d0003b`), zero new features.** A full-codebase architecture audit arrived from another
session; its findings were verified rather than accepted, and both directions moved._

**1. 🔴 The audit's headline finding was overstated, and proving it took one test.** It filed 7 High
findings on missing ownership anchors, all "UNVERIFIED", and named the verification as its own item 6.
Its sharpest claim — *"a signed-in client could call `saveClientWeight('<another clients uuid>')` from
devtools"* — is **false in practice**: as the E2E client against a different client's id, all four
inserts are refused by RLS specifically (42501) and the `clients` UPDATE matches zero rows. Two rows
downgraded High → Medium. **The test itself needed hardening twice** before it proved anything: it
could not distinguish an RLS refusal from a schema rejection, and it mutated a real client's
`goal_weight_kg` without capturing or restoring it — so a loose policy would have corrupted real data
while the test passed green.

**2. 🔴 My own escaping sweep shipped a worse bug than the one it fixed.** `escapeAttr` in a plain
`value=""` backslash-escapes, the browser hands `.value` back WITH the backslash, and
`saveSettingsProfile` writes it to `profiles.full_name` — compounding on every save
(`O'Brien -> O\'Brien -> O\\\'Brien`). Caught by pre-push review, verified empirically, fixed at all
3 sites (`9d0003b`). `app-clients.js:442` had used `escapeHtml` on that same field all along.

**3. `checks.sh` rule 9d was scoped narrower than the class it certified — twice.** First version
excluded any line containing `===`/`!==`, which let an unescaped `program.description` through on the
very run meant to prove it. Second omitted `.name` — the commonest free-text field here — so it would
have caught 6 of 17 sites and reported CLEAN while three live sinks sat in the tree. Now
`scripts/check-escaping.mjs`: per-MATCH extraction, markup-context aware, proven against all six
shapes. **A detector scoped narrower than its class is worse than none — it closes the ledger row.**

**4. ✅ The push gate is trustworthy again.** `runner.spec.js` clicked `Start`.first() 22 times —
"whatever workout is first" — which other tests mutate mid-run. Replaced with an `ownWorkout` fixture
(each test owns its workout; a client cannot insert a template, so it is created as the PT in a second
context). **`checks.sh` run three consecutive times: 56 passed, 0 failed, 0 flaky in every one.** Full
suite 382-387/6-9 flaky → **413 passed / 0-1 flaky**.

**5. ~350 rows of dead test data purged**, with Jake's go-ahead: 100 fake workout logs, 153 fake
templates, 100 fake programmes. That account held 156 templates of which **3 were real**. Zero real
rows touched, zero dangling references — the 2026-07-10 data-loss shape was explicitly guarded against.

**6. Shipped from the audit:** dashboard fetch errors are now visible (19 queries, none had an error
check — a failed load was indistinguishable from an empty account on the first screen of every login);
10 modals routed through `mountModal` (proven: a double-tap gave 2 overlays and the save read the
hidden one); Progress chart leak + 2 silent deletes; 20 escaping sites; a favicon, because a
permanently-red console trains you to ignore console errors.


**1. 🔴 Three live data-integrity bugs shipped, all of which made numbers WRONG rather than absent.**
(a) `assisted` recorded band-assistance as *lifted* weight into `weight_kg` — a −20kg assisted pull-up
would have stored as +20kg and fed volume, e1RM and PB detection. Unreachable via the UI, and a live
count found **zero** affected sets, so fix-forward with no repair (`d09db2b`). (b) A logged **RIR was
displayed as RPE** — opposite scales, so a brutal set read as trivial to any coach setting next week's
load (`5f892d3`). (c) **Captured HR/watts/pace/stroke-rate were discarded** whenever an interval had a
cool-down: the capture attached to the last logged row, which is the cool-down, and every Progress
aggregate filters cool-downs out by design. Silent at input, at save, and at the chart (`2c7ff09`).

**2. The coach and the athlete were reading different set counts off the same session.** A 4-round
interval wrapped in a warm-up and cool-down said **6 sets** in the runner + coach view and **4** in My
Progress. All four raw-count sites now call `_countableSets` itself rather than re-implementing the
predicate; warm-up/cool-down rows are NAMED and work rounds renumbered 1..N so the table agrees with its
own header (`40db93e`). **The first attempt was half-done** — it fixed the coach's screen and introduced
the same contradiction on the athlete's finish screen; caught by the pre-push review (`730c03f`).

**3. Five highest-damage silent writes closed, from a scan that found 21.** A failed clone returned a
valid id for an EMPTY workout; `duplicatePhaseWeek` saved the coach's week but not the client's and said
it worked; `deletePhaseWeek`'s 4-step renumber could half-fail and still say "Week deleted" (`d4b2689`).
**16 remain deliberately** — cleanup deletes whose worst case is an orphan row, plus `starter_seeded`.

**4. The runner's dead wizard deleted — 155 lines, and the class that made it reachable.** Only 133 were
wizard-only; `logRunnerSet` is shared with cardio. The real fix was routing: `_isPlainStrengthExercise`
was an allowlist of 5 metric types with the wizard catching "anything else", and *anything else was
reachable* because `_resolveMetricType` returns `metric_type` verbatim. Now `ex.type !== 'cardio'`, so
the table is the fallback. Verified ZERO wizard-bound rows live (53 templates, 105 logged) before cutting
(`262f092`). **Consequence: `supersetGroup` now has no reader — supersets are inert in the runner** until
the grouped-work slice.

**5. The pre-push review earned its place: 8 findings, 5 of them regressions THIS session introduced.**
Worst was reintroducing a toast-clobber bug documented 1,200 lines up in the same file from a previous
review — three error toasts painted over by the success toast, so the user sees only green. Also: the
half-done set-count fix; a silent-skip my own silent-writes fix made newly reachable; the routing change
dropping the prescription bar and 1RM banner; and rollback deletes that were themselves silent writes
(a surviving orphan reaches the GDPR export). All fixed (`4f23ce0`, `730c03f`, `c4e7ecb`).

**6. The bug ledger left STATUS.md.** 125 rows in a markdown table → `bugs/`, one file per bug with
frontmatter. That table shape had caused two real failures: unescaped `|` in prose split rows so os-lint
skipped them, and a Status cell at the far right of a 6,923-character line drifted from its own text for
17 days. Round-trip verified BEFORE the table was deleted (125 in / 125 out / 0 malformed); stale-bugs
read 12 before and 12 after. os-lint gained a `bug-files` check, proven RED→GREEN on a probe.

**1. 🔴 The health check had been lying, and is now mechanically prevented from lying again.** Six ledger rows
said `✅ FIXED + LIVE <commit>` in their **description** while their **Status cell** still said `open` — the
oldest fixed 2026-07-23, i.e. 17 days of a RED count inflated by already-done work. The real damage was alarm
fatigue: a check that is permanently red stops being read, which is how the genuinely-open rows got buried.
All six verified against their cited commits rather than trusted from the row text (e.g. `7fe41e0`'s commit
message never mentions the delete-button resize, so that one was confirmed by reading the actual diff) and
relabelled `fixed — awaiting Jake` — a relabel, **not** a close. `stale-bugs` 21 → **15**, every remaining row
genuinely open. **New `os-lint` check `ledger-drift`** catches this class permanently: flags any row whose text
claims fixed while the cell says open, deliberately excluding `HALF FIXED` rows and rows whose own text says
the fix was insufficient. Verified red→green (GREEN on the corrected file, RED after reverting one row, GREEN
after restoring). **`/save` Step 3a** now states plainly that writing the ✅ into the text is *not* updating the
status, and gates on `ledger-drift` being GREEN. Root cause was never discipline: the rule said to mark a fix
`fixed — awaiting Jake` but never said **which field**, and the ambiguity resolved the same wrong way every time.

**2. Predictions: 62 → 32 ungraded.** Graded 30 from evidence — **20 correct, 8 incorrect, 2 partial**. The
calibration finding is sharp and one-directional: **every "this fix will hold" prediction that could be settled
was wrong** (pth-071, -094, -085, -078, -096, -064, -122, -075), while **every "this will surface a problem"
prediction was right** (pth-053, -074, -080, -016). Confidence in stability is systematically overstated.
Of the 32 left, 17 are `owner` (only Jake can grade); most of the 15 `world` ones are unsettleable for one
honest reason — **the live verification they depend on has never happened**, and each maps to a ledger row
already sitting at `fixed — awaiting Jake`. Those two backlogs are really one.

**3. Conditioning runner teardown — the 2026-08-07 occlusion finding is genuinely fixed.** Drove a real
interval exercise end to end at 390×844 and looked at every screen. The rest timer is a compact **header bar**,
not a fullscreen overlay, so the page stays visible; the capture card appears after the block with
`overlay: false` asserted programmatically; erg targets (pace/500m, watts, HR zone, rest) all render on the
pre-Start card. Zero page errors. **One item needing Jake's eyes and only his:** the capture fields show the
prescribed targets as grey **placeholders** (145/160/210). Code-correct — `value` is empty, so
Continue-without-typing saves nothing — but this is the exact pattern Jake reversed once before ("a pre-filled
value is indistinguishable from one you actually entered"). Whether grey reads as *target* vs *already filled*
mid-session in a gym is a human judgement automation cannot settle.

**4. Strategy refreshed + a real competitor finding.** `coachapp-product-strategy.md` was 5 weeks stale and
**strength-biased**: a sweep of the entire research corpus for conditioning terms (erg/rower/SkiErg/Hyrox/
metcon/WOD/circuit/interval) returns essentially nothing — every source was read through a barbell lens, so the
strategy was written without conditioning ever being considered. Research pass run: **Trainerize's own idea
forum carries "Zone interval training / metcon cardio intervals" at 1,169 votes, still unresolved** (answered
Aug 2025 with a workaround video, not a feature); Hevy's cardio is duration + typed distance only. TrueCoach
probably cannot (low confidence); TrainHeroic and PT Distinction — **no evidence found either way**, searches
surfaced only their coaching *content*, not product capability. **Verdict: the conditioning wedge survives but
is half-evidenced — 2 of 5 checked convincingly.** Also recorded: the original "all features, no tiers" wedge is
owned by PT Distinction, and the client-experience wedge is now contested (FitFocus markets almost the same
sentence). The differentiators that remain are the two that came from Jake's own real use — conditioning, and
coach-as-athlete — neither of which the research ever saw.

**5. Jake ran the cardio metric_type retag SQL live** — 7 cardio machines (`Rowerg`, `RowErg`, `Skierg`,
`SkiErg - 10km SS`, `SkiErg - long/short intervals`, `Run`) were tagged `weight_reps`, so they routed to the
strength table and could not record duration/distance/HR/watts at all. Retagged to `cardio` in both `exercises`
and `workout_template_exercises`; verification returned 0/0. **Open follow-up:** commit `1379c05` then merged
`cardio` into `interval` as the sole cardio-family type, so those 7 rows may now be the only `cardio` rows left
— a read-only check is with Jake. Not harmful (the builder lands a legacy `cardio` row on "Intervals" rather
than falling through to weight_reps), but untidy. **Do not simply re-run the merge script** — its first
statement drops the original migration's backup table.

_Previous, 2026-08-05 (end of session) — **PUSHED to origin/master** (`3457d90..980d324`, 1 commit).
Picked 4 items off the kanban shortlist: the exercise-builder mobile grid restructure (built + shipped),
and 3 investigations (Workouts-page delay, new-template slow save, `workout_logs` fixture count) — all
measured live rather than reasoned about, per this project's Iron Law. Full suite 337 passed / 5
pre-existing `client-workout.spec.js` failures (documented baseline, unchanged) / 1 pre-existing flaky
(`runner.spec.js:400`, unrelated) / 2 skipped._

**1. ✅ SHIPPED — exercise builder set-editor now a 2-column grid on mobile (`980d324`).** Closes the
2026-07-22 report ("scrolls too much + resembles the Heavyset builder"). `renderTemplateSets`'s
weight_reps/unilateral, timed_hold and jump branches each had 4 always-visible fields (Reps/Weight/Rest/
Effort or their per-type equivalents) as full-width stacked rows — packed into a `.ts-grid` 2-column layout
(new CSS in `main.css`, matching the `.field-row` pattern already used elsewhere in the app), same "+ More
targets" disclosure kept for Intensity/Tempo/Countdown. Every input id, fallback expression and conditional
(bodyweight/assisted/AMRAP) is byte-identical to before — `flushTemplateSets` reads by id, not row
structure, confirmed by 119 targeted + 337 full-suite tests all still green. cardio/interval branches
untouched (already had progressive disclosure, not part of the complaint). Live preview built and approved
by Jake first (interactive HTML artifact recreating both layouts from the app's own real CSS tokens with a
live scroll-height measurement) before any code was touched. mobile-checked at 390×844 across all 3 set
types — screenshots confirmed no overflow/clipping, "+ More targets" still expands correctly. Multi-agent
review (3 angles) clean: zero queries/writes in scope (Agent A), zero role/solo branching touched (Agent
B), zero drift/dropped fields across the 3 restructured branches, verified field-by-field (Agent C).
Cache-bust: main.css v8→9, app-workouts.js v52→53. **Needs your eyes on your own phone.**

**2. Workouts-page delay + new-template slow save — MEASURED, no code bottleneck found (3rd investigation,
still open).** Both reports (2026-07-06 re-opened, 2026-07-13) share a suspected root cause per the
2026-08-02 note — checked properly this time with real timing, not more code reading. `renderWorkoutTemplates`
(the Templates tab render) and `saveNewTemplate → openTemplate` (create + display) were traced end to end:
one query and one query respectively, no N+1, no unbounded loops. Measured live in Chromium against the real
E2E account: **Workouts-page nav ≈242ms, template create+display ≈450ms** — both fast. This is the second
time the code itself has been checked and found clean (2026-08-02 static read, now a live timing measurement)
— the leading remaining candidate is the thing your OWN console already showed: Edge's "Tracking Prevention
blocked access to storage" (×12), which would silently delay supabase-js's own session-token read on *every*
API call, independent of query efficiency, and would explain a general "everything feels slightly slow"
complaint better than one inefficient query would. **Cheap experiment for you to run**: open the app in Edge
with Tracking Prevention temporarily set to Basic (or an InPrivate window), repeat the same navigation, see
if the delay disappears. I can't reproduce Edge's tracking prevention in this environment's Chromium, so this
needs your device. Left open — not closing on a guess a third time.

**3. `workout_logs` "fixture erosion" — REFRAMED, not what it looked like (2026-07-12, re-investigated
2026-08-02 and today).** The 2026-08-02 pass confirmed all 14 DELETE call sites are narrowly scoped (independently
re-confirmed tonight via a full grep of every `workout_logs` touch point across the test suite — every one
uses a specific id or a genuinely unique timestamped tag, no broad filters). **New finding tonight**: read
`scripts/seed-test-data.js` — it only ever inserts **5** "Push Day A" rows, gated to run once. The "13 → 4"
baseline this item has been tracked against was never that seed script's own count. A live query just now
found the E2E client's actual current `workout_logs` count is **154** — roughly 30 "Push Day A" rows (6× what
the seed script creates) plus dozens of distinct `[E2E]`-tagged names from many different spec files
(`trend`, `rec`, `mt`, `diary`, `col`, `1RM Check`, `Week-Label Session`, `Zero-Set Session`, `probe`, …).
This is **accumulation, not erosion** — debris from tests across many sessions that never got swept, not
rows disappearing. The old "was 13, now 4" observation was likely a real but transient snapshot (caught
mid-run, between an insert and its own test's later cleanup), not a standing downward trend. **Not cleaned
up tonight** — deliberately: a mass-delete against a shared, real Supabase account this deep into a session
is exactly the kind of destructive-action-under-momentum this project's rules warn against, and several of
these tagged rows may still be load-bearing for specific tests. Recommend a dedicated session to sweep the
identifiable `[E2E]`-tagged debris and reconcile "Push Day A" back toward what the seed script actually
creates, with the sql-safety discipline applied even though this is a test account, not real client data.

---

_Previous, 2026-08-02 (end of session) — **Everything that day was PUSHED to origin/master**
(`de54bdb..3457d90`, 24 commits total: the Solo-genuine-role feature pending push from 2026-08-01 below, the
previous session's 5 easy-win ledger items, and 5 new fix batches built and multi-agent-reviewed that day).
Confirmed via `git fetch` + `git log origin/master..HEAD` (empty) at end of session — local and origin are
byte-identical, nothing outstanding.

**That day's 5 batches, in order (all reviewed, all tested, all pushed):**

1. Diary/notes/ownership-anchor (`3728890`, review-caught test fix `2029fc7`) — see below.
2. Jump reps range + builder declutter (`393f1f6`) — see below.
3. **Box Jump wizard fix + PT/Personal boundary audit (`98b1b9b`)** — the runner's wizard-mode logging
   screen had no jump_height/jump_distance case at all (predates the metric_type system), so a jump
   exercise reaching it had nowhere to enter a height — fixed. Boundary audit closed out: `client_1rms` and
   `goals` confirmed already RLS-safe; `events` confirmed genuinely vulnerable (see #4).
4. **🔴 CRITICAL — `events` RLS gap found and fixed, confirmed by Jake live.** The `"coach access"` policy's
   `qual` (`client_id IN (your clients) OR created_by = you`, meant to cover client-less personal calendar
   entries) was being silently reused as the write-check too, so the self-authorship half of that OR let
   any coach insert an event against ANY client. Jake ran a targeted `ALTER POLICY ... WITH CHECK` live;
   re-ran the probe that caught it red→green, then independently verified legitimate coach writes
   (own-client + personal client-less events) still work.
5. **Mobile calendar CSS Grid fix + 3 more backlog items investigated (`3457d90`)** — the calendar grid
   genuinely broke on a long workout name (reproduced live via DOM injection before fixing: columns
   misaligned, days missing from view). Fixed with `min-width:0;overflow:hidden` on every day cell. Also:
   cardio set builder's "4 wrong counts" and the add-workout-picker/CSS-vars items from earlier today all
   turned out already fixed in prior sessions (ledger hygiene only); the 1RM 0.5kg-shift report and the
   `workout_logs` fixture-isolation bug were both re-investigated with no confirmed mechanism found and left
   honestly open rather than guessed at.

**Earlier batches, detail:**

1. **Diary/notes/ownership-anchor batch (`3728890`, review-caught test fix in `2029fc7`)** — diary now shows
   "No sets logged" instead of `0/0/0` tiles; client-authored per-exercise notes (typed in the runner,
   previously write-only) now render in `openWorkoutLog`; `saveEditTemplateExercise`/`deleteTemplateExercise`
   gained an app-level ownership anchor (`_verifyTemplateOwnership`) as defense-in-depth — a live 2-account
   probe had already confirmed RLS blocks the cross-tenant case even without it; `weight_logs` INSERT and
   `workout_template_exercises` UPDATE cross-tenant probes both confirmed already RLS-safe, with new
   permanent regression tests. Review caught the ownership-anchor test itself calling the real functions with
   swapped/null ids — passed for the wrong reason; fixed and re-verified red→green by temporarily neutering
   the anchor (the row stayed unchanged, proving RLS — not just the anchor — already blocked this).
2. **Jump reps range + builder declutter (`393f1f6`)** — jump height/distance exercises only ever prescribed
   a SINGLE rep count, not a range like every other set type, in 3 places sharing one 2026-07-22 design
   choice (builder, runner target bar, day-row text) — all 3 fixed together, Jake's live report. Also per
   Jake's live request: removed BW, Assist, and the round-Repeat control from the builder's per-set editor.
   Repeat had nothing persisted, so it and its now-dead `repeatTemplateSet` function + 2 tests were deleted
   outright. BW/Assist are stored per-set flags that affect other rendering, so their toggle stays visible
   only on a set that ALREADY carries the flag — a legacy escape hatch (same pattern as this file's existing
   "Pace / km (legacy)" row) so old data stays editable, but new sets can't acquire the flag via this control.
3. Full suite, final clean run: 328 passed / 5 pre-existing `client-workout.spec.js` failures (documented
   baseline — was tracked as 4, now confirmed 5; the 5th, "renders a hero card... Up next," was newly
   surfaced this session and independently reproduced against clean, already-committed HEAD before any of
   tonight's edits, proving it is NOT a regression from tonight's work — just another face of the same
   long-open embed-chain issue below) / 2 skipped. Cache-bust: app-progress v33→34, app-runner v49→51,
   app-workouts v50→52.

---

_Previous, 2026-08-01 (2nd save this date, + a live SQL follow-up same date) — **"Solo becomes a genuine
role" SHIPPED — merged to master locally (`1ef09c9`).** ✅ **PUSHED 2026-08-02 — see top of file.** Closes the
item 3 sessions overdue below ("Solo must become a genuine, independent account type," open since 2026-07-24,
deprioritized past beta 2026-07-28).

**Live SQL follow-up, same date:** Jake ran `scripts/migrate-solo-role-2026-08-01.sql` live in Supabase —
Step 1 diagnostic showed only `coach`/`client` roles existed (no surprise values, `ADD CONSTRAINT` safe to
run); Step 2 succeeded; Step 3 verify confirmed exactly 1 row now has `role='solo'` (id
`0ad8daff-bb11-4e7a-aff3-5d4413883f45`, `solo_only` still `true` as designed, `starter_seeded` already `true`
— seeded once before, back when this was still a normal coach account pre-lockdown, so its existing content
is likely still tagged `is_personal:false` and may be invisible to this account's own reads now). **Live
login check to confirm — not yet done, Jake said "let's move on for now."** If it turns out empty, the fix is
a small targeted `UPDATE` flipping just that account's starter-content rows (the ~40 named exercises,
"Example — Full Body A", "Example — 4-Week Foundation") to `is_personal:true` — not a re-seed, not touched yet.

**What shipped, scope exactly as approved (brainstorm → spec → plan → 4-task SDD build → final whole-branch
review → merge):** `profiles.role = 'solo'` is now a real, permanently-stored DB value for the one real
`solo_only` account, instead of an in-memory reassignment `loadUserInfo()` computed fresh every login from
the `solo_only` boolean. Master accounts (one login, both a coach identity and personal training) are
explicitly untouched — verified byte-for-byte identical. Public signup for solo accounts is deliberately
out of scope, deferred to its own future conversation per Jake's own call during brainstorming.

- **`scripts/migrate-solo-role-2026-08-01.sql`** (NOT YET RUN — this is tomorrow's Step 1): diagnostic →
  widens `profiles.role`'s CHECK constraint defensively (drop-then-recreate inside a transaction, guarded by
  a role-distribution check) → flips the real `solo_only=true` account to `role='solo'`, but **only if** it
  actually has its self-referential `clients` row (added after the final review flagged the original draft
  had no such guard) → verify. `solo_only` column deliberately left in place, unused as a steady-state
  signal, but now ALSO doubles as a transitional safety net (see below).
- **`js/app-core.js` `loadUserInfo()`** — removed the dead `solo_only`-checking branch, replaced with a
  branch keyed on `role === 'solo' OR (role === 'coach' AND solo_only === true)` — the OR clause exists
  specifically so the code is safe **regardless of whether the SQL migration has run yet**: this project
  auto-deploys on push, and the SQL is a separate manual step, so the code had to tolerate either order. A
  first cut of this only widened the condition, not the body — the final review caught that the account
  would render as a full (locked-out) coach dashboard pre-migration; fixed by reassigning
  `currentProfile.role = 'solo'` inside the branch too, verified with a dedicated red→green test.
- **`js/starter-content.js` seeding fix** — this is the actual BUG this feature exists to fix (found by the
  earlier 2026-08-01 full-file review, documented below): the seeder's own gate checked the wrong role value
  and every seeded artifact was hardcoded `is_personal: false`, permanently invisible to a solo account's own
  reads. Both fixed; `isSoloAccount` uses the same transitional OR-check, guarded by `!window._masterAccount`
  so a master account's temporarily-reassigned display role never gets mistagged.
- **3 pre-existing tests genuinely broken by this branch, fixed not papered over**: `tests/solo-only-2026-
  07-24.spec.js` hard-coded the internals of the retired `solo_only` reassignment mechanism (setting the
  flag directly, regex-matching old source text) — updated to test the new mechanism's equivalent safety
  properties (verified the underlying guarantees — no coach-view escape hatch, no seeding into `is_personal:
  false` — still hold, they just moved to different code).
- Cache-bust: app-core v7→v8, starter-content v3→v5.
- Full suite (final, clean, post-merge-verified-byte-identical): **311 passed / 5 pre-existing unrelated
  `client-workout.spec.js` failures (documented baseline, unchanged) / 1 known flaky (passed on retry) / 2
  skipped.**
- Design spec: `docs/superpowers/specs/2026-08-01-solo-genuine-role-design.md`. Plan: `docs/superpowers/
  plans/2026-08-01-solo-genuine-role.md`. Built via Subagent-Driven Development in an isolated worktree — 4
  tasks, each with its own red→green test and task-scoped review; a final whole-branch review caught one
  Critical (the deploy-ordering gap above) and 3 Important findings (migration missing a clients-row guard;
  non-transactional constraint drop-then-add; a stray untracked debug spec inflating the reported test count
  by 1), all fixed and independently re-verified before merge.

**Still needs Jake, in order — this is the actual Task 5 of the plan:**
1. ✅ **Done, same date** — ran `scripts/migrate-solo-role-2026-08-01.sql` live: Step 1 diagnostic reviewed,
   Step 2 succeeded, Step 3 verified `role='solo'` on the one real account.
2. ✅ **Done 2026-08-02** — pushed `master` to `origin` (was local-only, several commits ahead).
3. Log into the real `solo_only` account and confirm: lands on the solo dashboard with no errors, starter
   content actually appears (~40 exercises, one example workout, one example program), and specifically that
   the seeded exercises show `is_personal: true`, not just that they appear. **Not yet done** — Jake said
   "let's move on for now" when asked. Given `starter_seeded` was already `true` before this migration ran
   (this account was seeded once, correctly, back when it was still a normal coach account pre-lockdown, as
   `is_personal:false`), there's a real chance this step reveals the old content is still invisible — the fix
   for that specific case would be a small targeted `UPDATE` on just this account's starter-content rows, not
   a re-seed (the seeder only runs when `starter_seeded` is false, which it no longer is).
4. Only then does this ledger row close for real, per the closure rule below.

---

_Previous, same date, earlier save: 2026-08-01 (1st save) — **WEEKLY FULL-FILE REVIEW (overdue 8+ days) found
a real, previously-undetected stored-XSS cluster in the runner — fixed same session, plus a confirmed (not
speculative) solo_only seeding bug documented for a dedicated fix** (the exact bug the entry above went on to
fix). Continuing the same overall session as 2026-07-30 below —
Jake approved the proposed 3-item plan ("go ahead") after the CRITICAL `workout_logs` fix: (1) security
follow-up — done, see 2026-07-30 entry; (2) weekly full-file review — this entry; (3) Solo account type
decision — done above.

Ran `multi-agent-review` in **full-file mode** (whole files, not a diff) against the 2 highest-churn
modules (`app-runner.js`, `app-workouts.js` — 44/43 commits in 30 days, well ahead of the rest). Scoped to
2 modules instead of the usual 2-3 given how much ground the session had already covered. The 3rd review
angle (duplicates/render-safety) hit an API spend limit mid-review, same as one agent did earlier in the
session — substituted with a lighter direct check (duplicate function-name grep across all 9 modules:
clean; `setInterval`/`clearInterval` count ratio in both files: clears outnumber sets 17:7, no leak
signal). The other 2 angles completed in full and both returned real findings.

**🔴 FIXED — a genuine stored-XSS cluster in the runner's prescription-rendering path, ~15 sinks across
`_buildTargetCols`/`_renderTargetBarHtml` (the shared target-bar builder — feeds BOTH the table and wizard
render modes), the cardio target-chip row, several wizard-branch input `placeholder=`/`value=` attributes,
and the PT-note block.** `sets_json`/`notes` are coach-authored JSONB with no schema enforcement — every
one of these had an escaped sibling elsewhere in the same file, proving this was drift, not a design
choice: `_buildTargetCols` escaped `tempo` only (one field out of ~6), leaving reps/%1RM/effort/rest raw;
the PT-note block's twin in `app-workouts.js` already escaped all 3 of its branches. Confirmed direction:
coach-authored data renders unescaped to whoever runs that session in the gym — at minimum a compromised
or malicious coach account attacking their own clients (this app's own GDPR/health-data context makes that
a real threat model, not a hypothetical). The other direction (a client writing to their own plan clone's
`workout_template_exercises`, which would be the client→coach shape that's hit this codebase 3+ times
before) needs a live probe to confirm or rule out — not done this session, flagged below. Fixed by escaping
every raw field at the point it's built (matching the existing `tempo` precedent) rather than escaping at
the shared render sink, since some values reaching that sink are already-escaped or safely-numeric-formatted
and double-escaping would corrupt them. 3 new red→green tests in
`tests/full-file-review-2026-08-01.spec.js`. Cache-bust: app-runner v47→48, app-workouts v47→48.

**Also verified via live 2-account probes, all confirmed already RLS-safe (no fix needed):** `client_1rms`
INSERT with an untrusted `client_id` (same shape as the CRITICAL `workout_logs` gap, same file, ~100 lines
away — reasoned-risky, tested clean); a client attempting to INSERT into `exercises` claiming an unrelated
coach's `coach_id` (not their own real coach) — also blocked.

**✅ ALREADY FIXED, CONFIRMED 2026-08-09 — this was closed same-day, a few hours after this row was
written.** `fab4945` (2026-08-01 18:19) fixed both halves: `_seedStarterContent`'s own gate widened to
accept `role==='coach' OR 'solo'` (app-core.js's trigger already did), and all 4 `is_personal` write/read
sites switched to a shared `isSoloAccount` flag instead of a hardcoded `false`. `9357c31` (same day,
18:37) closed a reviewer-found follow-up: `isSoloAccount` now also requires `!window._masterAccount`, so
a master account whose display role gets reassigned to `'solo'` via the view-switcher never has its real
shared coach content mis-marked `is_personal:true` on a later seeding retry. `tests/solo-genuine-role-
2026-08-01.spec.js` (4 tests, re-run 2026-08-09) — test 3 explicitly asserts `_seedStarterContent` writes
`is_personal:true` for a `role='solo'` account, still green. This row was simply never closed out when
the fix landed — ledger hygiene gap, not a missed fix. Re-verified live-code-current, not just historical,
while checking this before onboarding a new solo beta-tester account. — (orig) **🔴 CONFIRMED (not
speculative — traced the exact execution path directly), NOT FIXED this session —
a `solo_only` account's starter content can never actually be seen, for two independent reasons.** (1) A
role-check race: `js/app-core.js:303` decides whether to seed based on the RAW db role (correct — its own
comment says so, "regardless of which branch above reassigned the display role"), but
`js/starter-content.js:84` re-checks the LIVE `currentProfile.role`, which the branch at
`app-core.js:255-275` has already reassigned to `'solo'` by the time seeding would run — so seeding
silently never happens, and `starter_seeded` never flips, so this repeats forever. (2) Deeper than the
race: even if seeding ran, every artifact is written with `is_personal: false`
(`js/starter-content.js:93` and siblings), but a `solo_only` account's role is ALWAYS `'solo'` for reads,
and every read path in scope filters `.eq('is_personal', currentProfile?.role === 'solo')` — meaning
`is_personal: false` content is structurally invisible to this account regardless of the race. The
starter-content seeder needs to know it's seeding for a `solo_only` account and flip `is_personal: true`
for that case specifically — a real, multi-part fix (seeder + role-check), not a quick patch, deferred to
its own session given how much ground tonight already covered. Affects exactly one real live account today
(the `solo_only` family-member account) — worth a live check on that account specifically before the next
fix attempt, not another guess.

**✅ PARTIALLY FIXED + PROBED 2026-08-02.** Two of the three sibling functions now have an app-level
ownership anchor: `saveEditTemplateExercise`/`deleteTemplateExercise` (`app-workouts.js`) gained a shared
`_verifyTemplateOwnership(templateId, coachId)` helper (checked before write; the update/delete itself now
also filters `.eq('template_id', targetId)` and checks the returned row count), matching the
anchor-AND-verify pattern every other sibling in the template family already used. Live 2-account probe
confirmed the OLD code path was already RLS-safe (an unrelated coach's attempt to mutate another coach's
template exercise via the real app functions was a silent no-op even before this fix) — this is hardening
in depth, not a closed vulnerability. Red→green test in `tests/ledger-fixes-2026-08-02.spec.js` (PT2
attempts both functions against PT's real template exercise; both confirmed no-op post-fix). **Multi-agent
review caught a bug in the test itself, fixed same session**: the first draft called
`saveEditTemplateExercise`/`deleteTemplateExercise` with swapped/null ids, so both calls short-circuited
before ever reaching `_verifyTemplateOwnership` — the test passed for the wrong reason. Fixed by correcting
the argument order and injecting the minimal modal DOM/globals these functions read from. Re-verified
red→green by temporarily neutering `_verifyTemplateOwnership`: the test still passed, because RLS itself
already blocks the cross-tenant write — confirming this fix is genuine defense-in-depth, not the only thing
standing between an unrelated coach and this row. `saveExerciseToTemplate`
(the INSERT path) and the `_propagateExerciseChangeToTemplates`/`_checkSiblingPropagation` fan-out remain
unanchored — not re-touched this pass. Cache-bust: app-workouts v50→51. — (orig) **Full-file review, not
fixed, flagged for later:** `workout_template_exercises` writes (INSERT/UPDATE/DELETE across
`saveExerciseToTemplate`/`saveEditTemplateExercise`/`deleteTemplateExercise`/the
`_propagateExerciseChangeToTemplates` fan-out) carry no app-level ownership anchor, unlike every sibling in
the template family (`saveEditTemplate`/`deleteTemplate`/`saveEditExercise`/`deleteExercise`/
`toggleExerciseArchived` all anchor AND verify row count). The fan-out's own `program_phases` lookup
(`_checkSiblingPropagation`) is unanchored too. Not directly reachable through the UI — reasoned as
RLS-only-defended, not proven — same "reasoned isn't proven" gap this session's other 3 probes closed for
their own tables. A handful of `x?.coach_id || currentUser.id` silent-fallback reads still exist
(`app-workouts.js:1589/1595/1929`, `app-runner.js:2676`) — read-paths only (safe direction: shows your own
data instead of theirs), except `:1589`/`:1929` feed onward into an `exercises` INSERT, worth re-checking
together.

Previous, same session: 2026-07-30 — **3 "FIXED" BUGS FROM 2026-07-29 WERE STILL BROKEN — real root causes found,
review surfaced 9 more sibling instances of the same falsy-zero class, all fixed same session (not yet
pushed).** Jake opened the session with the exact repro path for the add-exercise bug plus two one-line
reports ("cannot add 0 to depth jump", "depth jump does not show any previous exercise history"). All three
were things the *previous* session had already marked "fixed — awaiting Jake" — so this was root-cause
archaeology, not a fresh bug hunt: each 2026-07-29 fix was real but landed one layer above the actual defect.

**1) Add-exercise-not-appearing (4th report).** The 2026-07-29 fix (`_afterTemplateExerciseSave`) correctly
closed the propagation-chain failure it targeted — it just never ran. `saveExerciseToTemplate` closed the
modal (`closeModal` really removes the DOM), then re-read `#att-notes`/`#att-superset` a SECOND time from
the now-deleted modal to build `window._lastExerciseChange.row`, throwing `Cannot read properties of null`
and aborting before the re-render call. The insert had already gone through, so the exercise really was
added — it just never repainted without a manual refresh. Fixed by capturing both fields once, before
`closeModal` runs.

**2) 0 rejected for Depth Jump height.** Same falsy-zero mistake as 2026-07-29's weight fix — Jake's report
proved the class extends past weight. `toggleTableSet`'s jump guards, `_syncLoggedSetsFromTable`'s jump
branches, and `saveRunnerSession`'s height_cm/distance_m writes all treated JS's falsy `0` as "nothing
entered." Fixed with the same `_hasNumVal` helper (renamed from `_hasWeightVal` — no longer weight-specific).

**3) Depth Jump last-session history still not showing.** A controlled live experiment (log a real jump set
via the actual save path, relaunch, inspect the fetch) proved the pipeline fixed 2026-07-29 genuinely works
end-to-end for ordinary data — so the report meant a DIFFERENT gap. Multi-agent review found it while
auditing #2 for siblings: `fetchRunnerLastSession`'s own set filter (`s.weight_kg || s.reps_achieved ||
s.height_cm || s.distance_m`) is a truthy check — a jump set logged with `height_cm: 0` and no reps is
`0 || null || 0 || null` = excluded, even though it saved correctly. Very likely the actual mechanism.

**Multi-agent review then found 9 more instances of the same falsy-zero class**, none reported by Jake
directly, all fixed same session: a sibling `if (s.weight)` bug in `saveWorkoutSession` (the manual Log
Session modal — a completely separate save path from the runner); the SAME modal's own weight input
re-rendering a real "0" back to blank when a set is added afterward (would have silently defeated the
`saveWorkoutSession` fix in normal use); `_setDetailsLine` (My Progress diary) dropping a real 0cm jump;
`editRunnerSet`'s edit-overlay pre-fill blanking a real 0kg set; and `openWorkoutLog`'s past-session viewer
showing "—" for a real 0kg set. **Deliberately NOT fixed, flagged for a scoped follow-up**: `fmtDistanceM`
has the identical bug shared across ~15 cardio/jump-distance call sites — too broad to fold into a
height-focused session. Also surfaced, pre-existing and unrelated to tonight's diff: a `clients` id-only
read with a swallowed-error coach_id fallback in both save paths (needs a behavioural cross-tenant probe,
not yet confirmed exploitable), session names in `log.*` calls, and an unanchored `exercises` UPDATE.

Root-caused each bug via direct code reading and controlled live Playwright experiments against the real
Supabase backend (not guesses) before writing any fix, per this project's Iron Law. `tests/ledger-fixes-
2026-07-30.spec.js` — 10 tests, every one red-first verified (reverted the touched files to HEAD, confirmed
each assertion failed, restored). Full suite: 292 passed / 5 pre-existing unrelated `client_programs`-
fixture-gap failures (same tracked class as prior sessions, confirmed untouched by this diff) / 2 skipped.
Cache-bust: app-workouts v46→47, app-runner v46→47, app-progress v30→31.

**Same session, round 2 — Jake said "fix the bug findings," meaning the items flagged above as deliberately
deferred. Turned up one CRITICAL, confirmed-exploitable finding — NOT YET CLOSED, needs Jake to run SQL.**

**🔴 CRITICAL — `workout_logs` INSERT policy has no ownership check between `client_id` and `coach_id`.**
Confirmed exploitable via a live 2-account probe: `PT2` (an unrelated coach, owns zero clients) successfully
inserted a `workout_logs` row with `client_id` pointing at PT's real client — first through the app's own
JS (a sloppy fallback, `clientRecord?.coach_id || currentUser.id`, defaulted to PT2's own id when the
ownership lookup was correctly denied by RLS), **then confirmed again via a RAW insert that bypassed the
app's JS entirely** — proving the gap is in the RLS policy itself, not just careless app code. Any
authenticated coach can currently fabricate a workout session against any other coach's client if they know
(or guess) that client's UUID, and the real client sees the fabricated session in their own history. Fixed
the JS half (`saveRunnerSession`/`saveWorkoutSession` now refuse to save and show an error instead of
defaulting `coach_id`, red→green tested). **The RLS half needs Jake**: `scripts/fix-workout-logs-insert-
policy-2026-07-30.sql` — a diagnostic-first, self-contained script per this project's sql-safety
convention. **Caught a real bug in the fix's own first draft before it ever reached Jake**: an unqualified
`coach_id` inside the policy's `exists()` subquery silently bound to `clients.coach_id` instead of
`workout_logs.coach_id` (Postgres resolves unqualified names against the innermost FROM first, no
ambiguity error across query levels) — which would have rejected **100% of solo's own workout saves**
while leaving the actual coach_id-attribution check vacuously true for everyone else. Found independently
by 2 of 3 review agents; fixed by table-qualifying the column. **Still open, flagged but not investigated
further this session**: `workout_log_exercises`/`workout_log_sets` may share the same shape (a bogus
`workout_logs` row can't be created anymore once the fix lands, but existing real logs could theoretically
be re-pointed via an UPDATE policy that hasn't been checked); `workout_logs`' own UPDATE/DELETE policies
weren't audited either.

**Also fixed, same round:**
- **PII sweep, 19 sites total across 4 files** (`app-runner.js`, `app-workouts.js`, `app-progress.js`,
  `app-calendar-goals.js`) — session names, exercise/template names, check-in values, goal/event/milestone
  titles stripped from `log.*` calls, ids/dates/counts kept. The pre-push hook's own PII regex only matches
  explicit `{ name: ... }` syntax, not ES6 shorthand `{ name }` — it would not have caught any of these; a
  manual sweep found them all. Missed 6 of the 19 on the first pass (calendar/goal/milestone titles) —
  caught by the same review round that found the CRITICAL SQL bug.
- **`fmtDistanceM` (app-workouts.js) had the same falsy-zero bug** flagged (not fixed) in round 1 —
  `if (!n) return ''` treated a real 0m the same as blank. Fixed. **Review then found a regression the fix
  itself introduced**: `_cardioDistanceM(s)` returns a literal `0` (never null) for "not entered" by design
  — every other caller sums/compares it as a number — so 2 unguarded display call sites started rendering
  "0 m" on every duration-only cardio set (the common case). Fixed with an explicit `> 0` check at both
  sites, matching the guard pattern already used at every OTHER `fmtDistanceM(_cardioDistanceM(...))` call
  site in the same file.
- **`openWorkoutLog`'s past-session viewer got a jump_height/jump_distance display branch** — previously
  had only cardio vs. weight/reps, so a past jump session showed nothing meaningful for height/distance.
  Known incomplete: `saveWorkoutSession` (the manual Log Session modal) never writes `metric_type` on its
  `workout_log_exercises` rows the way `saveRunnerSession` does, so a jump session logged through THAT
  modal specifically still won't render under the new branch — not a regression (that modal has no jump
  entry UI today), just a latent gap if it ever gets one.
- **6 more unanchored writes found and fixed**: `toggleExerciseArchived`/`saveEditExercise`/
  `deleteExercise`/`_rememberExerciseMetricType` (`exercises`, anchored on `coach_id`) and `deleteEvent`/
  `deleteGoal` (anchored on `created_by`). A live PT2 probe against the `exercises` writes found these were
  **already blocked at the RLS layer** even before the JS anchor — hardening, not closing an active hole
  (unlike `workout_logs`). `deleteEvent`/`deleteGoal` were not individually probed (reasoned from the
  existing `events(created_by)`/`goals(created_by)` FK-anchor convention and confirmed both are only
  reachable from coach-only UI) — **their siblings weren't touched**: `saveGoalProgress`, `saveEditGoal`,
  `toggleMilestone`, `toggleClientMilestone` are the same id-only shape, still open, added to the ledger.

Multi-agent review (3 angles) ran twice this session — once per round. Round 2's third agent was cut off by
an API spend limit mid-review; had 2 of 3 full reports plus a partial third, judged sufficient given both
completed agents independently found and confirmed the same critical SQL bug (strong convergent signal).
Full suite after all round-2 fixes: 300 passed / 4 pre-existing unrelated `client_programs`-fixture-gap
failures / 1 flaky-unrelated (`solo-account.spec.js`'s Library nav test, not touched by this diff, passed
on retry) / 2 skipped. `app-calendar-goals.js` was edited but its cache-bust was initially missed — caught
before push. Cache-bust final: app-workouts v46→47, app-runner v46→47, app-progress v30→31, app-calendar-
goals v5→6.

Previous: 2026-07-29 — **6 LIVE RUNNER/BUILDER BUGS SHIPPED LIVE** (`eb9ec3f`, CI green, pre-push hook
56/57 smoke green). Jake dropped beta-timeline pressure explicitly ("push it back if needed, don't worry
about beta") and handed over 8 things found using the app for real — 6 in one message, 2 more mid-review.
Investigated via 3 parallel Explore agents + direct reads, planned via a Plan agent (Plan Mode, approved
before any code), built, then the mandatory `multi-agent-review` (3 angles) caught **3 more real bugs in the
fixes themselves** before push — all fixed same session.

**Shipped:**
1. **0 weight silently rejected, then silently dropped even after the reject was lifted** — `toggleTableSet`'s
   `!row.weight` guard (and 3 sibling sites: `_syncLoggedSetsFromTable`, both `saveRunnerSession` weight_kg
   writes) treated JS's falsy `0` as "nothing entered." A bodyweight-only load typed as literal "0" was
   blocked at the UI, and even past that would have saved as `null`. Fixed all 4 sites with an explicit
   not-null/not-empty check (`_hasWeightVal`).
2. **Jump exercises (Depth Jump etc.) never showed last-session data** — `fetchRunnerLastSession`'s
   `select()` never included `height_cm`/`distance_m` (stale allowlist, same class as les-036), so a jump
   set's history was silently unfetchable even though it existed. Widened the select + the ghost-text
   fallback.
3. **Add-exercise-not-appearing-until-refresh, reported a 3rd time, finally root-caused** (see ledger row
   below, was open since 2026-07-13). `saveExerciseToTemplate`/`saveEditTemplateExercise`/
   `deleteTemplateExercise` each ended in a bare, un-awaited, uncaught propagation call — any failure inside
   it (network blip, an RLS path behaving differently on a real account) silently left the stale pre-edit
   list on screen. New shared `_afterTemplateExerciseSave` re-renders immediately and toasts on failure.
4. **"Bodyweight" removed as an exercise Category option** — redundant with the real per-set BW toggle;
   implied a whole exercise was bodyweight-only when that's a per-set choice. Dropped from both dropdowns +
   `starter-content.js` seed data (existing tagged rows left as-is, fix-forward).
5. **Rest timer redesigned to survive navigating to view another exercise mid-rest** — Jake's own framing:
   "fix as a bug or redesign." Decided: the countdown (incl. beeps/voice cues) now keeps running in the
   background while viewing a different exercise, with a persistent "Resting — 0:45" chip to jump back,
   instead of being torn down by any navigation. New `_runner._restForExIdx`/`_restPendingFire` fields
   decouple "a rest is counting down" from "is it the one on screen." The pre-existing regression test
   guarding the corruption class this redesign had to avoid reopening (`intervals-redesign-2026-07-25.spec.js`
   line 411) was re-run and confirmed still green, not just assumed.
6. **Same-day program assignment "not there" — reproduced live, root cause was NOT the date.** Assigning a
   program with **zero phases** (e.g. one just created, not yet built out) silently created a `client_programs`
   row with zero `client_program_workouts` and no error — the exact "assigned, then not there" symptom, with
   no date logic involved at all. `_cloneProgramForClient` now fails loud (toast) on this and on a genuine DB
   error, instead of a bare silent `return`.

**Multi-agent review then caught 3 more real bugs in the fixes themselves, all fixed same push:**
- The solo self-assign flow's own unconditional "Program added to your plan" success toast was instantly
  overwriting fix #6's new warning (this app's `showToast` keeps a single DOM node, no queue) — for solo
  specifically, the new fail-loud toast was invisible for exactly as long as it takes the next line to run.
  Fixed by having `_cloneProgramForClient` return a success flag the caller now checks first.
- Fix #3's restructure introduced a genuine race: `_afterTemplateExerciseSave` re-read `window._templateCtx`
  fresh AFTER an `await` (a real network round-trip inside `openTemplate`), so a coach clicking into a
  DIFFERENT client's plan during that gap could have their edit's propagation silently write onto the wrong
  client's templates. Fixed by snapshotting `ctx`/`window._lastExerciseChange` before the await and threading
  them through explicitly.
- "⇄ Swap exercise"/"+ Add exercise" aren't gated off during a rest and bypassed fix #5's new nav-persistent
  logic entirely — could orphan a frozen floating overlay whose Skip button was still wired to fire against
  whichever exercise is now on screen (not the one the rest belonged to). Fixed on both paths (add preserves
  the rest + drops the stale overlay; swap of the resting exercise itself abandons the rest cleanly), plus a
  defensive ownership check added to `skipRestTimer()` itself.

**Not built — flagged for a scoping conversation, per Jake's own split of the ask:** per-exercise unit
override (new: force one exercise to a fixed unit regardless of the account toggle) and supersets-redesign
vs. WOD/circuit-training (confirmed as two separate asks, not one).

14 new tests (`tests/ledger-fixes-2026-07-29.spec.js`), all red-first verified against pre-fix code (reverted
the file to HEAD via `git show`, confirmed the assertion failed, restored). One pre-existing test in
`intervals-2026-07-24.spec.js` was rewritten from a source-text regex to a real behavioral test (multi-agent
review flagged the regex as provable-but-not-enforced). Full suite: 287 passed / 4-5 pre-existing unrelated
`client_programs`-fixture-gap failures (same tracked class as prior sessions) / 2 skipped. Cache-bust:
app-runner v45→46, app-workouts v45→46, app-programs v26→27, starter-content v2→3.

Previous: 2026-07-28 — **INTERVALS REDESIGN + A MAJOR SECURITY SWEEP SHIPPED LIVE** (34 commits from edb8995
to 60238be, CI green). Two things happened this session, in sequence.

**1) Intervals became a real exercise type.** The old model was structural: each interval round was a duplicated
row in `sets_json`, which is why the builder needed an awkward "Repeat ×N" cloning control and the runner used a
one-shot callback chain (`_afterRest`) that couldn't express nested structure. Jake's own Tabata Stopwatch Pro
reference screenshots (shared mid-session after a live "no intervals visible" report turned out to be a display
question, not a missing-feature one — the actual gap was the block-editor UX and the runner model) drove the
redesign: **one block** in `sets_json` (`countdownSecs`/`warmupSecs`/`workSecs` or `workDistanceM`/`restSecs`/
`sets`/`recoverySecs`/`cycles`/`cooldownSecs`) describing the whole workout, expanded by a pure function
(`_expandIntervalBlock`) into a flat phase list (`countdown → warmup → [(work→rest)×sets → recovery]×cycles →
cooldown`) that the runner walks by index (`_startPhaseAt`/`_advancePhase`, returning continued-vs-finished so
callers know whether to re-render). Decisions locked with Jake before building: identical rounds only (no
ladders/pyramids), work measured by time OR distance, warmup/cooldown recorded not just timed, rest/recovery
timed-only. New `workout_log_sets.phase` column (`warmup`/`work`/`cooldown`, NULL for every ordinary set) lets
Progress-page aggregates exclude warmup/cooldown from set counts and volume (`_countableSets`, `!s.phase ||
s.phase === 'work'` — the `!s.phase ||` half is load-bearing, every pre-existing row is NULL). Migration
`scripts/add-interval-type-2026-07-25.sql` (metric_type CHECK extended, new `phase` column + its own CHECK) —
run live by Jake before any code task started (les-053 gate). Built via Subagent-Driven Development, 9 tasks,
each with a fresh implementer + dispatched review + fix-loop; **8 real bugs were caught and fixed across the
build**, several found independently by two different reviewers reasoning from different angles (strong
corroboration signal): a total runner freeze on every interval workout (a function call wired before its
definition existed — missed by 20 tests because none let a real timer reach zero); a phantom set silently
written to the database whenever the get-ready countdown was tapped through early, shifting every subsequent
set number in that session; the runner repainting over its own finish screen on the last phase of the last
exercise (same failure class this file has hit before); a stray-timer leak when jumping between exercises
mid-rest; a silent no-op where the warmup/cooldown exclusion filter would have done nothing because the
underlying query never selected the new column; mid-session swap/add showing "Set 1 of 1" for an 8-round block;
and the "Log session" manual-entry modal silently discarding an interval block's entire prescription into one
blank row. Full interval-feature suite: 49 tests, all red-first verified.

**2) A live, unrelated, CRITICAL stored-XSS was found and closed mid-review.** The mandatory whole-branch
`multi-agent-review` (3 pinned angles + verifier, required before any push per CLAUDE.md) flagged
`client_check_ins.notes` — a client's own free-typed weekly wellness note — rendering completely raw on the
coach's client-profile Overview tab, the first thing a coach sees opening a client. Confirmed live on
production, pre-existing (the three prior stored-XSS fixes in this codebase's history — 2026-07-13, -18, -23 —
covered `performance_logs`/`weight_logs`, never this table), unrelated to intervals in every sense (different
file, zero code overlap). Told to Jake plainly; he said fix everything found, then push — twice more, as the
review chain kept finding siblings: client `email`/`phone`/`notes` (3 separate render locations each — one of
them proven, not just suspected, client-writable: `saveWeightGoals` updates `clients` with no `coach_id`
filter, and a client session was shown live to be able to write its own `email`/`phone` through that same RLS
grant), goal/milestone `title`/`description`, an event `title` on the coach's own calendar (`saveClientEvent`
proves a client can create their own event, stamped `is_pt_assigned:false` specifically to mark it
client-authored), and — in a final consistency pass — 9 more self-XSS-tier sinks (dashboard goal/milestone
widgets, workout template description) that the earlier rounds had left half-fixed. ~33 sinks closed in total
across 3 commits. **Also found along the way**: the new `client_check_ins` regression test's own cleanup
silently no-ops — that table has no DELETE grant for either role via the client API (a blocked delete returns
success with 0 rows affected, not an error), so 7 debris rows (inert — the escaped payload text, harmless)
had already accumulated in the shared Test Client fixture. Documented rather than hidden; Jake has the one-line
cleanup SQL. Same accepted-debris class as the already-tracked `workout_logs` fixture-erosion issue below.

**Solo slipped past its 31 July beta target** — see the bug ledger row below for the full timeline; the short
version is Jake chose to build intervals over resuming Solo when the two collided this session, and intervals
ran long enough (a security incident included) that nothing further happened on Solo either.

Final state: 279 declared (full run) = 273-275 passed / 2 skipped / 0 failed, run alone / 4-5 recurring failures
that are a confirmed pre-existing, unrelated `client_programs` test-data gap (the shared Test Client fixture's
program assignment is empty — same class as the tracked `workout_logs` erosion issue, not a regression from
this push — confirmed directly via a live DB query, and confirmed no code in any of the 34 commits touches
`programs`/`client_programs`/RLS). Cache-bust: app-core v=7, app-dashboard v=6→8, app-clients v=8→9,
app-programs v=24→26, app-calendar-goals v=2→5, app-workouts v=35→45, app-runner v=34→45, app-progress v=27→30,
starter-content v=2. Previous: 2026-07-25 (2nd save) — **PLAYWRIGHT VIEWPORT BUG FIXED LIVE** (edb8995, CI green). Every mobile
check this project has ever run was silently executing at desktop width (1280px), not the intended 390px —
`playwright.config.js`'s `chromium` project spread `...devices['Desktop Chrome']`, whose own viewport overrode
the top-level mobile default. Fixed by re-overriding viewport after the spread, which unmasked the real bug it
had been hiding: the app's sidebar nav and bottom nav both carry an identical `data-page` attribute for every
page (CSS alone decides which is shown, via a 900px breakpoint), so a bare `page.click('[data-page="x"]')`
resolved to the sidebar copy and timed out once the viewport was genuinely mobile. Same shape for the Personal
view-switcher and sign-out — sign-out turned out to have a third wrinkle, a second non-viewport-gated button
living in the Settings page body. Added `clickVisible`/`waitForVisible` to `tests/helpers.js` and migrated 56
call sites across 19 spec files, sequenced call-sites-first (proving the swap alone changed nothing) then the
config flip. At the real 390px viewport: 218/220 passed, 2 skipped, 0 failures, 1 flake that reproduced clean
in isolation — **zero new real mobile bugs surfaced**, meaning the app itself was already fine and only the
tests were desktop-coupled. Multi-agent review (3 angles) clean. Pure test-infrastructure — the only product
code touched was one inert `id` attribute (app-progress v26→v27). Session flow: user asked "what is next
proposal" after the previous save's kanban shortlist; picked this item via AskUserQuestion over the banked
Solo-account-type decision (deliberately left for a scoping conversation, not a straight build); root-caused
via 3 parallel Explore agents (confirmed the 2026-07-23 ledger analysis's mechanism was still correct but every
cited line number had drifted from 2 days of commits since), designed via a Plan agent, then built+shipped.
Previous: 2026-07-25 (1st save) — **UNITS TOGGLE SHIPPED LIVE** (23a2493, CI green). Account-wide, per-metric-type unit
preference — weight (kg/lb), jump height (cm/in), cardio distance (km/mi) — confirmed as Jake's real, current need
(himself: kg for weight, inches for jump height, simultaneously), which is why this is 3 independent toggles, not
one global metric/imperial switch. Storage stays canonical (kg/cm/metres) everywhere in the DB; conversion happens
only at display/entry boundaries via shared helpers (`weightToPref`/`weightFromPref`/`fmtWeight`,
`jumpHeightToPref`/`jumpHeightFromPref`/`fmtJumpHeight` in app-core.js; `distanceToPref`/`distanceFromPref` extending
the existing `fmtDistanceM` in app-workouts.js), driven by `window._unitPrefs` from three new `profiles` columns
(migration `scripts/add-unit-prefs-2026-07-24.sql`, run live by Jake). Jake chose "everything, fully" over a partial
pass specifically to avoid a half-converted state — this rewired every prescription/entry/history display it
touches: the builder set editor (incl. cardio distance, a spot with a pre-existing header/unit mismatch bug fixed
along the way), the runner (fast table, wizard, manual "log a past session" builder, all three parallel 1RM-entry
flows, post-session estimate modal, session-review detail), the whole Progress page (diary, trend charts, personal
records, weight tracker + goals + chart axes), the program-assignment %1RM checklist, both bodyweight-log forms, the
PERF_CATEGORIES manual PB form (now defaults its unit dropdown to the account's preference), and a new Settings
"Units" card. **Multi-agent review caught 5 real issues before push, all fixed and re-verified green**: a
post-session "new 1RM estimate" modal was missed entirely (still hardcoded kg); four separate sites silently dropped
forced one-decimal display for kg-native users (`Math.round(v*10)/10` pre-rounding is NOT the same as the old
code's `.toFixed(1)` — a whole-number value now rendered "100" instead of the "100.0" every kg-only user had always
seen) — fixed by adding a `decimals` option to `fmtWeight`; and the mile-conversion constant `1609.344` was
triplicated inline instead of a shared name, the same drift-risk class this whole feature exists to prevent
elsewhere. **Also found and fixed, mid-session: a real test-fixture-pollution bug**, not a product bug — a Playwright
cleanup step signed out of the PT account and into a client account first, then tried to reset the PT's own
`weight_unit` from the CLIENT's session; RLS on `profiles` is self-scoped, so that write was silently rejected (not
just skipped), leaving the shared PT fixture permanently stuck on `weight_unit='lb'` after every run of that one
test and cascading into unrelated failures across `day-row-prescriptions`, `progress-trend`, and `programs.spec.js`.
Fixed by re-establishing the PT session before the reset. Separately swept 6 leaked `[E2E] 1RM Check/Have Program`
fixture rows + 36 leaked templates off the shared PT/client accounts (pre-existing debris from `programs.spec.js`'s
"Assignment-time 1RM check" tests never wrapping their cleanup in try/finally — a failed assertion before cleanup
left the fixture behind forever; both tests hardened). 220 declared = 218 passed / 2 skipped / 0 failed / 0 flaky.
New `tests/units-2026-07-24.spec.js` (6 tests: formatter/parser round-trips + blank/null contract, a completeness
grep-net for leftover hardcoded unit literals, a real builder round-trip through lb without corrupting canonical
kg, `saveSettingsUnits` persistence, and cross-role independence — a client's own preference is unaffected by their
coach's). app-core v6→v7, app-dashboard v5→v6, app-programs v23→v24, app-clients v7→v8, app-workouts v34→v35,
app-runner v33→v34, app-progress v25→v26. Previous: 2026-07-24 (3rd save) — **SECURITY: public self-signup closed, solo_only lockdown SHIPPED LIVE** (57a188a, CI green). Jake's brother self-registered via the public "Sign up" form and got a full PT/coach account with zero gatekeeping — signup removed entirely (code-side; Jake still needs to flip the matching Supabase Auth dashboard setting). Jake then asked for the brother's account to be locked to personal-only with no path to a coach dashboard — built `profiles.solo_only`, and in the process **also discovered and fixed a live incident**: an in-flight edit collided with a live migration and corrupted a real coach test account's role to 'client' in the production database (root-caused, repaired, and the underlying fallback logic hardened so it can't happen again — les-053). Multi-agent review then caught 3 more real bugs in the solo_only lockdown itself before push (unconditional role force with no fallback, stale session state surviving sign-out as a working escape hatch, fail-open to coach nav on a fetch error) — all fixed and tested. **Known, banked limitation**: solo_only enforcement is client-side only for now; a real DB-level fix is tied to the already-banked "Solo becomes a genuine account type" pre-beta decision. 214 declared = 212 passed / 2 skipped / 0 failed / 0 flaky. Previous: 2026-07-24 (2nd save) — **INTERVALS: get-ready countdown + builder repeat-set×N SHIPPED LIVE** (7aeb3ae, CI green). Jake described the flow he wants (Start → countdown → work → beep → rest → beep → next round → repeat); most of the loop already existed (`startIntervalTimer`/`startRestTimer` already auto-chain via `_afterRest`, both already speak the last 5 seconds aloud). Built the one real gap — a 5-second spoken "5,4,3,2,1,Go!" lead-in before the first work interval — plus "Round N of M" labeling, plus (per Jake's explicit ask when scoping the builder side: "shouldn't have to click add>copy 10 times") a one-click `repeatTemplateSet` that clones a set N-1 more times, type-agnostic so it also helps 5×5 straight sets. **Multi-agent review caught a real bug in the fix itself**: the new count-in timer wasn't wired into the same "block LOG / full teardown" guards the pre-existing rest timer already has — `logRunnerSet` could have silently logged an un-performed round mid-countdown, and `showRunnerFinish` could have left the count-in ticking under the finish screen. Both fixed same push. 204 declared = 201 passed / 2 skipped / 1 flaky (unrelated pre-existing runner-launch race), 0 failed. Previous: 2026-07-24 — **6-BUG LEDGER BATCH SHIPPED LIVE** (b637e09, CI green). Legacy metric_type fallback dead code, unbounded Epley 1RM estimate, "PB" badging the most recent entry not the best, 4 distance-format bypasses, Discard-button no-confirm, and cardio LOG silently no-opping — all fixed with red→green tests. **Multi-agent review caught a real bug in the fix itself**: the new best-PB picker compared raw numbers across incompatible units (100kg vs 220lbs — the bigger RAW number, 220, is actually the lighter lift) — fixed with a small unit-normalization table before any push. 197 declared = 195 passed / 2 skipped / 0 failed. Session also confirmed with Jake: the metric/imperial toggle has a REAL current need (him — kg for weight, inches for jump height, simultaneously) and is **account-wide, per-metric-type** (not the single global toggle previously assumed) — weight (kg/lb), jump height (cm/in), cardio distance (km/mi) in scope now; jump distance deferred until he hits that friction too. Not yet built — next up this session, alongside intervals. Previous: 2026-07-23 (2nd save) — **DAY-ROW PRESCRIPTIONS + PICKER DISAMBIGUATION SHIPPED LIVE** (b53dbfc, CI green). Day rows across all four surfaces now show what the plan actually prescribes (`4 × 8–10 reps · 100kg · RPE 8 · 2:00 rest`) instead of a bare set count, and the Add-workout picker tells you WHICH of two identical workouts you're picking (`↳ Used in Wave 1 · Wk 2 · MON`). Both built on newly-shared helpers that replaced silently-drifted duplicates. **The pre-push review found 5 blocking issues — 3 of them regressions the merge itself introduced** (les-048: merging drifted copies destroys exactly the drift that justified deduping). 181 declared = **179 passed / 2 skipped / 0 failed / 0 flaky**. ⚠️ **Also found: every "mobile check" this project has ever run was at 1280px, not 390px** — the chromium project's `devices['Desktop Chrome']` overrides the config's mobile viewport. Ledgered, not yet fixed. **Still open and needing Jake:** the Programs-page add-exercise report was driven live in both cases and NEITHER symptom reproduced — needs his actual program. Earlier same day: exercise-builder overhaul (c72eb14). Previous: 2026-07-19 (progress overhaul + analytics, 95e8e8f)._
---

## Live state

**App version:** app-core v=11 · app-dashboard v=11 · app-clients v=11 · app-programs v=40 · app-calendar-goals v=13 · app-workouts v=65 · app-runner v=69 · app-progress v=41 · starter-content v=5 — **all pushed and live as of 2026-08-14 (`d337418`).**
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

## Build history snapshot

| Version | What shipped |
|---|---|
| app-programs v=11 | Fixed a bare `class="btn"` Cancel button in the phase-form (no CSS definition, same bug class already fixed on the dashboards) — swapped to `.btn-secondary`. Cosmetic only, no logic change. **Pushed 2026-07-08 (6620720).** |
| app-workouts v=16 | Exercise-picker modal height synced to `window.visualViewport` instead of a plain `vh` unit — fixes the residual shrink Jake still saw once the on-screen keyboard opened (vh doesn't track the keyboard-shrunk viewport on most mobile browsers). Self-reviewed (no subagent review, explicit call given a tight usage budget this session); `runner.spec.js` 26/28 passed cleanly (2 flaky, pre-existing login-timeout race). **Pushed 2026-07-08 (b1aa50c).** |
| app-runner v=16 | Fixed solo mode landing on a broken "fetch failed" screen after finishing a workout — `_afterRunnerSave` only special-cased role `'client'`, so `'solo'` fell through to a PT-only `openClient()` call scoped by `coach_id = currentUser.id` (solo's own record has `coach_id = NULL`). Found via code review, not a Jake report. New Playwright regression test; multi-agent review (3 angles + verifier) clean. **Pushed 2026-07-08 (298d88d).** |
| app-workouts v=15 | Fixed exercise picker modal shrinking/drifting toward the bottom of the screen as search results narrow (`max-height`→fixed `height:70vh`). Verified live at 390×844 (constant height across query states). Full Playwright suite green (69 passed). **Pushed 2026-07-07 (682f86f).** |
| app-workouts v=14 / runner v=15 | Batched the N+1 per-exercise save loop in `saveRunnerSession`/`saveWorkoutSession` into 2 batched inserts each (measured 14→4 requests, 4.7s→1.1s on a 6-exercise save); added rollback to `saveWorkoutSession` (never had one); added `.limit(100)` to both Workouts-page queries; cleaned up 103 confirmed-orphaned `workout_templates`. Found + fixed a real stray-character syntax corruption mid-session (traced via live browser console, not guessed). 3-agent review caught + fixed 1 real gap (new test's cleanup depended on the rollback it was testing). 69/69 Playwright. **Pushed 2026-07-07 (444d0f3).** |
| tests/helpers.js only, no app version change | Found + fixed the real root cause of the test-suite flakiness blamed on "system fatigue" in the row below: `loginAsClient` never waited for the client dashboard's async render to finish before tests proceeded, unlike `loginAsPT`. Verified stable across 3 full runs of the 38-test pre-push smoke set. **Pushed 2026-07-06 (31698fe).** |
| app-programs v=10 / progress v=5 / runner v=14 / workouts v=13 | Exercise identity linking — real `exercise_id` FK replaces free-text name matching on `workout_log_exercises`/`client_1rms`; 4,777 template exercises/27 logged/18-19 1RMs migrated; new shared Exercise Picker replaces the old dropdown in 4 places; 5 bugs found + fixed via multi-agent review. **Pushed 2026-07-06 (1526704).** |
| app-dashboard v=3 / workouts v=12 / runner v=13 / progress v=4 | Session 17's backlog (false-positive save error, swap/add rest time, %1RM table routing, cardio preview fields, dashboard program header, `.modal-box`→`.modal` fix) pushed; review caught + fixed 2 more real bugs (weight-goals form unreachable from client/solo My Progress; orphaned partial workout_logs row on retry). 69/69 Playwright. **Pushed 2026-07-06 (9b1fb9c).** |
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
