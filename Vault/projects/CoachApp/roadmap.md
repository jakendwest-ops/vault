# CoachApp Roadmap

_Last updated: 2026-06-28_

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
| Client profile (tabs) | ✅ Done | Overview / Goals / Workouts / Weight / Performance / Programs / Photos |
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
| Custom exercise demo videos (YouTube link) | 💡 Future | Per exercise in library |

### Workout runner
| Feature | Status | Notes |
|---|---|---|
| Real-time gym logger | ✅ Done | |
| Cardio mode + interval timer | ✅ Done | |
| Rest timer | ✅ Done | |
| Bodyweight / assisted / superset support | ✅ Done | |
| Timed sets | ✅ Done | |
| Unilateral L/R logging | ✅ Done | |
| Set X of Y header + edit logged sets | ✅ Done | |
| Session summary / finish screen | ✅ Done | PR badge, stats row, exercise cards |
| Runner target chips (pace, duration, stroke rate, rest HR) | ✅ Done | |
| Client notes per exercise | ✅ Done | Saves to workout_log_exercises.client_notes |

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
| Goals overhaul — granular mini-goals and milestones | 🗓 Planned | Week 3 priority |

### Programs (phase-based training plans)
| Feature | Status | Notes |
|---|---|---|
| Programs schema (4 tables) | ✅ Done | |
| Programs UI — create / edit / delete programs | ✅ Done | |
| Program phases (weeks, ordering) | ✅ Done | |
| Assign workout templates to phase days | ✅ Done | |
| Assign program to client with start date | ✅ Done | |
| Edit start date | ✅ Done | Shifts entire calendar |
| Remove program from client | ✅ Done | |
| PT client programs accordion (Phase → Day → SESSION N/M → exercises) | ✅ Done | |
| Client plan editing (PT edits client's sessions) | ✅ Done | Apply-to-all propagation |
| Auto-create calendar events from program schedule | 💡 Future | |
| Progression rules engine | 💡 Future | Auto-calculates target weight per session |
| Individual session skip/move on client calendar | 💡 Future | Defer until real PT usage data |

### Branding / UI
| Feature | Status | Notes |
|---|---|---|
| Branding — logo upload, display on dashboards | ✅ Done | v151 — private logos bucket, signed URLs, sidebar + PT/client dashboards |
| UI consistency pass | ✅ Done | SESSION N/M labels + exercise lists unified across all surfaces (v143) |
| Metric / imperial toggle | 🗓 Planned | Week 3 priority |

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

_Jake's master account gets a third "Personal" pill in the view switcher. A solo user is their own coach: `coach_id = auth.uid()`. Existing coach-scoped and client-scoped RLS covers them with minimal new policies._

**Agreed design (2026-06-29):**
- Third pill: PT | Client | Personal in both desktop sidebar + mobile switcher
- Solo nav: Dashboard, Workouts, Programs, Calendar, Progress — no Clients section
- One-time SQL clone: Jake's existing client record data copied into new self-owned record (coach_id = auth.uid()), then independent — no link to PT account
- Program "assign to client" defaults to self in solo view
- Existing Jake West client record under PT account stays as-is (still appears in PT dashboard stats)
- **Decided (2026-06-29):** Delete the Jake West client record from the PT account after clone — PT account should only have real/test clients (Alex Turner, Sarah Mitchell, Test Client). Keeps the test environment clean.

| Feature | Status | Notes |
|---|---|---|
| Third pill — Personal view in master account switcher | 🗓 Planned | `switchView('solo')` — full session required |
| Solo nav + dashboard | 🗓 Planned | Client shell + self-create access |
| Self-coached client record (coach_id = auth.uid()) | 🗓 Planned | One new RLS insert policy on clients table |
| One-time data migration SQL | 🗓 Planned | Clone 8 tables from existing Jake West client record |
| Program self-assign flow | 🗓 Planned | Assign to self in solo view without client picker |
| Solo weight + PB tracking | 🗓 Planned | Already works for clients — minimal extra work |
| Solo workout runner | 🗓 Planned | Already works for clients — likely zero extra work |
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
| Pre-push git hook + GitHub Actions CI | ✅ Done | 10 checks; blocks bad pushes |
| Playwright E2E suite | ✅ Done | 14 tests passing; runner + client + auth flows |
| SQL safety skill | ✅ Done | |
| Vault → GitHub backup | ✅ Done | jakendwest-ops/vault; auto-pushed on /save |

---

## Beta prep (Week 5 — Jul 22–31)
- Full walkthrough, Playwright suite, Supabase redirect URL audit
- Beta invites staggered: Jul 25, Jul 28, Jul 31
