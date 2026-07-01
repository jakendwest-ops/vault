# CoachApp Roadmap

_Last updated: 2026-06-29 (session 3)_

---

## How to read this

Each feature has a **status tag**: `✅ Done` / `🔧 In progress` / `🗓 Planned` / `💡 Future consideration`
Features inside a section are in priority order. Update status tags during each `/save`.

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
| 1RM exercise-name matching — move from free-text to exercise-library-linked (exercise_id) entry | 🗓 Planned | Big design decision, deferred to its own scoping session. Currently `client_1rms.exercise_name` is free text with no exercise_id — matching is exact-string (trim+lowercase), same limitation the runner already has. Today's build uses one shared name-matching helper so swapping in exercise_id-based matching later is a contained change, not a rebuild. |
| Progression rules engine | 💡 Future | Auto-calculates target weight per session |
| Individual session skip/move on client calendar | 💡 Future | Defer until real PT usage data |

### Branding / UI
| Feature | Status | Notes |
|---|---|---|
| Branding — logo upload, display on dashboards | ✅ Done | v151 — private logos bucket, signed URLs, sidebar + PT/client dashboards |
| UI consistency pass | ✅ Done | SESSION N/M labels + exercise lists unified across all surfaces (v143) |
| Progress tabs — pill grid (no scroll) | ✅ Done | v171 — flex-wrap pills, all 5 visible at once on mobile |
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

_Jake's master account gets a third "Personal" pill. Solo user is their own coach: `coach_id = auth.uid()`. RLS covers them via existing coach-scoped + client-scoped policies._

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

## Beta prep (Week 5 — Jul 22–31)
- Full walkthrough, Playwright suite, Supabase redirect URL audit
- Beta invites staggered: Jul 25, Jul 28, Jul 31
- **Before beta:** 1RM system built + tested (drives %1RM in programs)
- **Before beta:** Delete old Jake West client record from PT account
