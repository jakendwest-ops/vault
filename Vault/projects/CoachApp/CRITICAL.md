---
bi_temporal: true
---

# CoachApp — CRITICAL facts

_Always-relevant facts. Changes append to the timeline block at the bottom — never overwrite._

---

## Infrastructure

| Fact | Value |
|---|---|
| Supabase project ID | `avilxuiacmtgeoxxhfhc` |
| Supabase region | eu-west-1 (Ireland) — adequate for UK GDPR |
| GitHub Pages URL | https://jakendwest-ops.github.io/coachapp |
| GitHub repo | jakendwest-ops/coachapp |
| Vault backup repo | jakendwest-ops/vault |

---

## Storage buckets

| Bucket | Visibility | Expiry | Notes |
|---|---|---|---|
| `logos` | Private | 604800s (7 days) | PT logo upload; MIME-restricted to image types; policies scoped to `auth.uid()` |
| `progress-photos` | Private | 3600s (1hr) | Client body photos. **Feature UI removed 2026-07-12** ("for now"); bucket + data retained. Now exactly 2 correctly path-scoped policies (client↔own `clients.id` folder, coach↔own clients' folders). |

**Rule: private is NOT sufficient — object policies must be path-scoped too.**
On 2026-07-12 `progress-photos` was `public = false` yet had 3 `storage.objects` policies scoped by
`bucket_id` alone (`"Public read"` SELECT, `"Authenticated delete"` DELETE, `"Authenticated upload"`
INSERT). Result: any *authenticated* coach could read/delete/write **any** client's photos — a live
cross-tenant leak, reproduced (a second coach downloaded a real 1.79MB photo and deleted another).
Dropped all three (`scripts/fix-storage-rls-2026-07-12.sql`); proven closed by
`tests/storage-privacy.spec.js`. A `select public from buckets` check passed this leak clean — only
the behavioural probe caught it. See `breach-procedure.md` §6.

---

## GDPR compliance status (2026-06-28)

| Requirement | Status |
|---|---|
| Consent at signup | ✅ Checkbox, validated in handler |
| Data export | ✅ `downloadMyData()` in Settings |
| Right to erasure | ✅ `delete_current_user()` RPC + Settings UI |
| Data residency | ✅ eu-west-1 (Ireland) |
| Privacy policy page | ✅ `/privacy-policy.html` live (v158, 2026-06-29); consent checkbox links to it |
| ICO breach notification | ✅ Documented — `breach-procedure.md` (72h rule, decision tree, mandatory log, worked example). 2026-07-12 |

---

## Security constraints — non-negotiable

1. **No PUBLIC storage buckets** — use signed URLs always
2. **No PII in log calls** — IDs and dates only; pre-push hook enforces
3. **RLS on every table** — enabled before first INSERT; never `qual = 'true'` on sensitive tables
4. **No `auth.users` in RLS policies** — use `auth.uid()` / `auth.email()` only
5. **Health data = special category** — weight, body fat, photos, performance metrics
6. **New table checklist** — RLS + add to `downloadMyData()` + add to `delete_current_user()` RPC

---

## Key DB functions

| Function | Purpose |
|---|---|
| `delete_current_user()` | GDPR erasure — deletes all user data then auth.users row; security definer |

---

## Timeline

- 2026-06-28: GDPR hardening session. progress-photos made private. PII stripped from 16 log sites. Consent checkbox added to signup. Data export + delete account built. Storage bucket RLS established for logos and progress-photos.
- 2026-07-01: Found and fixed a solo-account RLS gap — `client_1rms` had no INSERT/UPDATE/DELETE policy for solo users, and `client_programs` had no UPDATE/DELETE policy for solo users (both only had coach-scoped write policies + a partial solo insert/select). Added 5 solo-scoped policies matching the `client_program_workouts` pattern. Lesson: when adding a new role/account type, every table it touches needs its own explicit write policies checked — a working INSERT does not imply UPDATE/DELETE also work.
- 2026-07-03: GitHub Pages deploy source switched from legacy branch-deploy to Actions-only (`build_type: workflow`), fixing a redundant dual-workflow setup where GitHub's own native Pages workflow was auto-deploying independently of and alongside the repo's custom "Check & Deploy" Action on every push — both could independently hit GitHub's transient deploy-infra errors, producing duplicate failure emails for a single hiccup. Confirmed FK cascade rules for `programs`/`program_phases`/`program_phase_workouts`/`workout_templates`/`client_programs`/`client_program_workouts` while researching a separate fix: `programs→program_phases→program_phase_workouts` and `workout_templates→workout_template_exercises` cascade automatically on delete; `client_programs→client_program_workouts` cascades too (0 orphaned rows found live); `workout_templates` itself does NOT cascade-delete when its `program_phase_workouts` row is removed (SET NULL) — the one real gap, now handled explicitly in `deleteProgram()`.
- 2026-07-03: coachapp GitHub auth hardened. A live `ghp_` PAT was found (and accidentally surfaced in-chat) embedded in plaintext in coachapp's `.git/config` remote URL; Jake deleted it, the embedded token was stripped from the remote (`git remote set-url` to the clean `https://github.com/jakendwest-ops/coachapp.git`), the stale cached credential was cleared, and git now authenticates to github.com via the `gh` CLI's keyring `gho_` OAuth token (`gh auth setup-git`; `repo` scope). No secret remains in `.git/config`; verified with an empty push (ec30ebf). The `vault.git` remote was already clean (no embedded token). Lesson banked: never run a command that echoes a known-embedded secret (`git remote -v`/`get-url`, `cat .git/config`).
- 2026-07-28: **4th instance of the client→coach stored-XSS pattern found and closed** — `client_check_ins.notes` (a client's own free-typed weekly wellness note) rendered completely unescaped on the coach's client-profile Overview tab, the first thing a coach sees opening a client. Confirmed live, currently exploitable pre-existing (a malicious note runs with the coach's own Supabase session — RLS cannot defend against a stolen JWT), found incidentally by the mandatory whole-branch review gating an unrelated feature push. Prior instances of this exact pattern: 2026-07-13, 2026-07-18, 2026-07-23 (`performance_logs`/`weight_logs`, never this table). Swept and closed ~33 sinks total across `client_check_ins`, `clients` (email/phone/notes), `goals`/`goal_milestones`, and `events` — full detail in STATUS.md/LOG.md 2026-07-28. **This is the 4th time this exact bug class has been found reactively rather than caught by a systematic sweep** — worth a standing lesson: any new client-writable free-text field needs its every render site checked against escaping at the time it's added, not discovered later by a security-focused review.
- 2026-07-23: **GDPR export was not a complete disclosure** (`7fe41e0`). `downloadMyData` was an `if/else` on role — `coach` exported clients/templates/programs ONLY, so a coach who also trains could never export their own weights, workouts, PBs, goals, events or 1RMs by any route. `client_check_ins` (Art. 9 special-category) was in **no** branch; `workout_logs` exported `name,date` only, so N sessions meant N empty headers and zero sets/reps/loads. Both halves now run ungated and independently; reads use `.in('client_id', …)`; the error-discarding `.single()` is gone. Red→green `tests/gdpr-export.spec.js` (3 tests, one pinning the `clients_user_id_idx` UNIQUE constraint). **Process lesson (les-049): the review's stated root cause was WRONG and was repeated before checking** — two agents claimed a master account holds two `clients` rows so `.single()` threw; `clients.user_id` is UNIQUE, so that failure mode is impossible.
- 2026-08-01: **5th instance of the stored-XSS pattern** — a cluster in the runner's prescription render path, found by the (overdue) weekly full-file review. `_buildTargetCols`/`_renderTargetBarHtml` (feeding both table and wizard modes), the cardio target-chip row, several wizard placeholder/value attributes and the PT-note block all rendered coach-authored `sets_json`/`notes` unescaped — ~15 sinks, **every one with an escaped sibling elsewhere, proving drift rather than an original omission**. Direction: coach → whoever runs the session. Red→green `tests/full-file-review-2026-08-01.spec.js`.
- 2026-08-12: **6th instance, and the first confirmed REGRESSION against a specific dated fix.** The full-codebase audit found unescaped free-text renders across 5 files. `js/app-programs.js:87` (`renderClientPrograms`'s `sessionSummary`) computes byte-for-byte the same expression as `js/app-workouts.js:642`, which correctly wraps it in `escapeHtml` — app-programs rendered it raw. The 2026-07-18 hardening pass documented this exact bug as fixed but named only two surfaces; this was a third with the identical pattern, never brought in. Also `js/app-runner.js:844` — the "Your notes" textarea rendered `ex.clientNotes` raw into `innerHTML`, and that value **round-trips through `localStorage`** via `_saveRunnerDraft`/`_loadRunnerDraft`, so a resumed draft replays the injection on a later visit rather than only on the original keystroke.
- 2026-08-11: GDPR consent capture + privacy policy — **`deferred` by Jake**, tracked in `bugs/2026-08-11-gdpr-no-consent-capture-and-no-privacy-policy.md`. Recorded here because a deferred GDPR obligation is still an obligation; it must not be lost because it was postponed.
- 2026-08-23 (OS v3): this Timeline had received **no append since 2026-07-28**, during which the stored-XSS pattern it explicitly tracks recurred twice more (2026-08-01, -08-12) and two GDPR items landed. The document's own "changes append to the timeline block" rule had no mechanism behind it. `os-lint`'s `doc-obligations` check now goes RED when a `bugs/` file matching a tracked security class is newer than this file's last write. **The 2026-07-28 entry's standing lesson stands and is now stronger: this class has been found reactively 6 times and never once by a systematic sweep at the moment a client-writable free-text field was added.**
- 2026-08-29: **7th instance of the unescaped-render pattern**, found by the weekly full-file review. `nextEx.name` reaches `innerHTML` raw at `app-runner.js:607→:613` and `:1685→:1703` (both fire during REST, unavoidable in a normal session) while the *same field* is correctly escaped at `:803` and `:814`; `programs.name` is raw at `app-workouts.js:3071` while its sibling six lines below is escaped. **Four renders of one field, two safe, two not.** Self-XSS today — `exercise_name` and `programs.name` are coach-authored and `tests/template-exercise-write-rls-2026-08-10.spec.js:138` proves a client cannot write them — so unlike instances 4-6 there is no client→coach path. NOT yet fixed; `bugs/2026-08-29-unescaped-db-strings-reach-innerhtml-at-3-sites`. **Note for whoever fixes it:** `scripts/check-escaping.mjs` ran clean over all three, so the checker must be shown RED on this syntactic form (`'Next: ' + ex.name` assigned to a variable, then interpolated) BEFORE a clean run means anything — that is exactly how the 2026-08-16 "class closed" claim was wrong about nine live sites.
- 2026-08-29: **A new recurring pattern named: readability is not ownership.** `saveRunnerSession` and `saveWorkoutSession` fetched the `clients` row and treated "I can read it" as "it is mine". `_verifyClientAccess` (`app-core.js:1105`) asserts `coach_id === uid || user_id === uid` and fails closed — and `app-core.js:1093` records that it was *"deliberately modelled on saveRunnerSession"*, yet neither save path was ever migrated onto the helper it inspired, while both 1RM writers in the same file were. **2 of 4 client-scoped writers had the strong check; now 4 of 4** (`fe58592`). Today the only thing refusing a cross-tenant insert was `workout_logs_insert_owned_client_only` — the database, not the app. **One site remains**: `_effectiveCoachIdForClient` (`app-workouts.js:1940`) still returns `currentUser.id` on an unreadable row (`bugs/2026-08-17-effectivecoachidforclient-swallows-an-rls-denial`, read-only path, still open). The rule going forward: **an ownership check must compare an owner column to `auth.uid()`, never infer ownership from a successful SELECT.**
- 2026-08-29: **Also fixed same-day — a guard that verified one id and wrote on another.** `_resolveEditableTemplateId` (`app-workouts.js:2923`) verified `tmpl.coach_id` and then repointed `.eq('id', ctx.phaseWorkoutId)`, an id no caller had checked. Now anchored `.eq('template_id', templateId)`. This is the **second** site of that shape; the named sibling `removePhaseWorkout` was fixed 2026-08-22 and this one was missed. `program_phase_workouts` still has **no cross-tenant RLS probe** in `tests/` — a grep finds only fixture inserts, never a refusal assertion.
- 2026-08-29: **A deferred RLS question from 2026-07-30 was never answered.** `scripts/fix-workout-logs-insert-policy-2026-07-30.sql:27-34` Step 1 asks whether `workout_log_exercises` and `workout_log_sets` independently anchor on log ownership, and says *"confirm rather than assume."* Nothing in `scripts/` or `tests/` records an answer, 30 days on. The app never depends on those SELECT policies (every read is bounded by an already-scoped id list), so the residual exposure is a direct console INSERT against a foreign `log_id` — the same shape as the 2026-07-30 bug, one table down. `bugs/2026-08-29-a-deferred-rls-question-from-2026-07-30-was-never-answered`.
- 2026-08-29 (correction, same day): the entry above was first written as the **6th** instance, copying the review agent's wording. **This timeline's own count already reached 6 on 2026-08-12**, so it is the 7th. Corrected here and in the bug row. Worth keeping visible because the instance NUMBER is the only thing making this a tracked recurrence rather than a one-off — an undercount quietly resets the evidence that the class keeps coming back. The count is a fact stored in two places (this timeline and each bug row), which is `feedback_two_fields_one_fact`: **read the timeline's last number before writing the next one.**
- 2026-08-29 (7th instance, CLOSED same day — `c1842f0`): **the class was 8 sites, not the 3 the review found.** Fixing `scripts/check-escaping.mjs` FIRST — a one-hop indirection rule (a free-text value assigned to a local, then interpolated raw) — found **six**, three of them in files the review never opened (`app-dashboard.js:118` `full_name` into an `<h1>`; `app-programs.js:169`/`:183` session names), plus **two I introduced myself the same day** (`escapeAttr` in a plain `value=""`). The rule was **measured before being given teeth**: name-keyed file-wide gave 2 false positives; adding function scoping silently DROPPED a real site (a name-only key let the second assignment overwrite the first); keyed by function+name it gives 6 candidates, all 6 real, 0 false. **Two checker defects surfaced on the way, both by measuring rather than reading: (a) CRLF — both loops split on `\n`, and the trailing `\r` broke the assignment match, so the rule found ZERO sites in a file where it now finds two; the ORIGINAL loop had the same latent weakness, so the pre-existing rules may have been under-reporting for as long as the working files have been CRLF. (b) `NOT_A_SINK` applied to a compound RHS dropped a line holding a real sink AND a `.length`.** Proven able to fail: a planted line-leading `const` + raw `${}` goes RED. **The first plant used an IIFE-scoped `const`, stayed green, and I nearly recorded "the rule is dead" — the plant was wrong, not the rule.** That limit is documented in the checker.
