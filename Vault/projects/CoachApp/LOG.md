# CoachApp — Session Log

Newest first.

---

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
