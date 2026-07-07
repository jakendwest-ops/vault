# CoachApp Roadmap

_Last updated: 2026-07-06 (session 18)_

---

## How to read this

Each feature has a **status tag**: `✅ Done` / `🔧 In progress` / `🗓 Planned` / `💡 Future consideration` / `🐛 Bug`
Features inside a section are in priority order. Update status tags during each `/save`.

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
| 13 | Personal > Workouts page shows 1RMs — redundant with Progress section | Low | Confirmed location | app-workouts.js:277-296. Ties to item 10's PBs/Performance restructure — remove once that lands. |
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
| Client profile (tabs) | ✅ Done | Overview / Goals / Workouts / Weight / Performance / Programs / Photos / 1RMs |
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
| **Runner redesign → Hevy-style table logger** (supersedes "set input pre-fill") | ✅ Done (v1) | 2026-07-02, pushed 6e6402a: all-sets-visible table (`SET · PREVIOUS · KG · REPS · ✓`), previous values pre-filled → 1-tap repeat, tap-✓ to complete, non-blocking rest timer. **v1 = plain strength only**; cardio/timed/unilateral/%1RM stay on the wizard — phase 2 for those is still 🗓 Planned. No paid gating. See STATUS "What's working" + "In progress" for the two known v1 gaps (superset auto-switch, bodyweight live-verify) |
| **Runner redesign phase 2** — extend table (or equivalent) to cardio/timed/unilateral/%1RM exercises | 🗓 Planned | Scope not yet defined; v1 deliberately excluded these to ship the strength-only case first |
| **Plate calculator (what to load on the bar)** | 🗓 Planned | Repeatedly requested by lifters on Reddit; small, self-contained; sits next to the weight field (2026-07-02 research) |
| **Improve workout-tracking visuals + underlying data model** | 🗓 Planned | Jake, 2026-07-04: wants a better look at how a workout is tracked in the runner, and to reconsider where/how that data is stored. Not scoped yet — needs a sounding-board session before build (what specifically feels wrong about the current table/wizard visuals). The "where it's stored" half is now split out into its own scoped item below (runner autosave). |
| **Runner session autosave (localStorage draft)** | 🔧 Scoped, not built | 2026-07-05: the "where it's stored" half of the item above, split out and scoped on its own. Decided approach — hybrid: localStorage-only draft now (checkpoint on every `renderRunner()` call + a 10s safety-net tick; key `_runnerDraft_<clientId>`; captures both `loggedSets` and the strength-table's parallel `tableRows`; same-day staleness cutoff; resume/discard confirm modal wired into `startWorkoutRunner()`; cleared inside the existing `discardRunner()`). A DB-backed draft (for cross-device/reinstall recovery) is deliberately deferred to a later pass. Full implementation plan exists (exact functions, hook points, Playwright test cases) — build was interrupted before any code was written, so next session picks up the plan directly rather than re-scoping. Fixes the 2026-07-04 live incident (runner freeze + forced reload wiped an entire in-progress gym session). |
| **Rest time not overwritten on swap/add** | 🐛 Bug (confirmed 2026-07-05) | Swap mode never reassigns `ex.restSecs`; add mode hardcodes 90s, ignoring the entered value. See session backlog Area 1 #2. |
| **App feels slow saving an updated workout, and navigating dashboard → Workouts page** | 🐛 Bug, not yet investigated | Jake reported 2026-07-06 directly on the kanban board (not yet root-caused — no code read this session). Needs profiling next session: check for N+1-style queries on save, unnecessary re-renders on navigation, or a missing `.limit()`/index on a large table. See STATUS.md open to-dos. |
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
| **Personal Bests / Performance merge** | 🗓 Planned, needs scoping | 2026-07-05: Jake — the two tabs are redundant. Move 1RMs into Personal Bests; repurpose Performance to track saved-workout-session progress over time (date completed + trend). Flesh out in a future session. See roadmap session backlog Area 2 #10. |

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
| **App-wide undefined CSS vars/classes — found 2026-07-05, not fixed** | 🗓 Planned | `var(--bg-accent)`/`var(--text-accent)`/`var(--surface-2)` referenced 52× across 7 files, never defined — same bug class as the dashboard-card fix above but app-wide. `.modal-box` (app-progress.js/app-runner.js) same pattern. `app-programs.js:672` has the same bare-`.btn` Cancel bug fixed elsewhere. Deliberately kept out of the dashboard-only pass — needs its own audit of intended styling per site before fixing. |
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
| **Personal account mirrors Client view exactly** | 🗓 Planned, needs scoping | 2026-07-05: Jake's own architecture statement — the only difference between Personal and Client should be no client/PT linkage. Confirmed real diffs exist today (`renderSoloDashboard` lacks the sudo/branding banners `renderClientDashboard` has; solo has a unique 4-tile stats row). Needs a dedicated session — ripples into the redundant-1RMs-on-Workouts-page and calendar-preview-parity items. See roadmap session backlog Area 3 #15. |

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

- **⚠️ Pre-beta gates — action before 31 July:**
  - **ICO breach-notification procedure** — still undocumented (CRITICAL.md marks it ❌)
  - **Delete old Jake West client record** from the PT account (see "Solo user" section)
  - **Supabase Pro upgrade** — unlocks leaked-password (HaveIBeenPwned) protection
- Full walkthrough, Playwright suite, Supabase redirect URL audit, `/deploy-check`
- **Beta invites: single date, 31 July** (previously staggered Jul 25/28/31 — simplified to one date)
- ✅ **1RM system built + tested** (done 2026-07-01) — drives %1RM in programs
