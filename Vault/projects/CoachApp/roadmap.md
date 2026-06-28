# CoachApp Roadmap

_Last updated: 2026-06-20_

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
| Client profile (tabs) | ✅ Done | Overview / Goals / Workouts / Weight / Performance |
| Edit client details | ✅ Done | Verified working post-deploy |
| Update client email (modal) | ✅ Done | Verified working post-deploy |
| Invite client via email | ✅ Done | Edge Function — stamps user_id + invited_at at send time |
| Resend invite | ✅ Done | State driven by invited_at |
| Client compliance tracking | ✅ Done | Sessions per client this week — colour-coded card on PT dashboard |
| In-app PT→client messaging / notes thread | 💡 Future | Supabase Realtime; simple thread per client profile |

### PT dashboard (Monday morning view)
| Feature | Status | Notes |
|---|---|---|
| Stats row (active clients, sessions this week) | ✅ Done | |
| Recent activity feed (weight + workout logs, last 7 days) | ✅ Done | |
| This week's sessions compliance card | ✅ Done | Replaces "Needs check-in" — colour-coded per client, capped at 6 |
| Goals due soon (next 14 days) | ✅ Done | |

### Calendar
| Feature | Status | Notes |
|---|---|---|
| Calendar page (monthly grid) | ✅ Done | Mon-start, coloured event dots |
| Month navigation | ✅ Done | |
| Add event (modal — title, date, type, client, notes) | ✅ Done | |
| Delete event | ✅ Done | |
| `created_by` bug fix in saveEvent() | ✅ Done | Fixed: added created_by: user.id to insert |
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
| Log workout against template | ✅ Done | |
| View workout log | ✅ Done | |
| Custom exercise demo videos (YouTube link) | 💡 Future | Per exercise in library |

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

### Programs (phase-based training plans)
| Feature | Status | Notes |
|---|---|---|
| Programs schema (4 tables) | ✅ Done | programs, program_phases, program_phase_workouts, client_programs |
| Programs UI — create / edit / delete programs | 🔧 In progress | v94 build started; needs testing + review |
| Program phases (weeks, ordering) | 🔧 In progress | |
| Assign workout templates to phase days | 🔧 In progress | |
| Assign program to client with start date | 🔧 In progress | "Assign to client" button built; not fully verified |
| Auto-create calendar events from program schedule | 💡 Future | |
| Progression rules engine | 💡 Future | PT defines rules per exercise (linear/wave/peaking); runner calculates target weight — eliminates need for per-week template clones |

### Leaderboards
| Feature | Status | Notes |
|---|---|---|
| Client leaderboard (most weight lost, heaviest squat, etc.) | 💡 Future | Low effort, high engagement — uses existing data |

---

## Client-facing features

### Client dashboard
| Feature | Status | Notes |
|---|---|---|
| Client login (invite acceptance — set password) | ✅ Done | |
| Client-facing dashboard (what the client sees after login) | 🗓 Planned | **Next priority** — RLS foundations now in place |
| Client logs own weight | 🗓 Planned | |
| Client logs own workouts | 🗓 Planned | |
| Client views own goals + progress | 🗓 Planned | |
| Client views own calendar | 🗓 Planned | |
| Client adds own PBs | 🗓 Planned | |
| Client video submission (form review) | 💡 Future | PT reviews and leaves feedback |

### Solo user (self-coached)
| Feature | Status | Notes |
|---|---|---|
| Solo signup flow (no PT invite needed) | 🗓 Planned | Role: solo; coach_id null |
| Solo dashboard | 🗓 Planned | Same as client dashboard but self-serve |

---

## Business / platform features

| Feature | Status | Notes |
|---|---|---|
| Branding / custom logo | 💡 Future | White-label for gym owners |
| Marketplace — sell programs to non-clients | 💡 Future | Long-term monetisation path; program schema already supports it |
| Assistant coach support (multi-coach) | 💡 Future | Relevant for gym/team settings |
| MFP / nutrition CSV import | 💡 Future | Macro tracking, actual intake vs targets |

---

## Infrastructure / tech

| Item | Status | Notes |
|---|---|---|
| Supabase project (avilxuiacmtgeoxxhfhc) | ✅ Done | |
| RLS policies — coach-scoped | ✅ Done | |
| RLS policies — client-scoped | ✅ Done | 11 SELECT policies; using `in` subquery pattern; auth.email() not auth.users |
| Broken policy fix (`Client stamps own user_id`) | ✅ Done | Was querying auth.users directly — replaced with auth.email() |
| Audit log (DB-side triggers on 17 tables) | ✅ Done | |
| Structured console logging (JS-side) | ✅ Done | log.info/ok/error/warn throughout app.js |
| Silent failure audit | ✅ Done | All error handlers now log + show inline; no alert() remaining |
| Edge Function — invite-client | ✅ Done | |
| Git (master branch) | ✅ Done | |
| Netlify deploy | ✅ Done | https://superlative-khapse-b92582.netlify.app (v=22 pending deploy) |
| SQL safety skill | ✅ Done | `coachapp/.claude/skills/sql-safety/SKILL.md` |
| Session protocols (hello claude / /save) | ✅ Done | |
| RLS extension for client accounts | ✅ Done | |

---

## Pending actions for Jake (manual steps)

| Action | Priority |
|---|---|
| Deploy v=22 to Netlify (drag files inside coachapp/ to dropzone) | High |
| Commit sql-safety skill to CoachApp git repo | Medium |
