# CoachApp — Product Blueprint

_Locked decisions. Do not change without Jake's explicit agreement._

## Product vision
A PT/coach management platform, built PT-first. Solo and client accounts added on top.
Goal: best-in-class UX for coaches managing 5–20 clients. Feature depth second, workflow speed first.

## Account types

| Role | coach_id | Description |
|------|----------|-------------|
| `coach` | own ID | Manages clients, builds programs, monitors all client data |
| `client` | PT's ID | Invited by PT; sees own data, logs sessions, adds personal events |
| `solo` | null | Personal training log only; no PT relationship |

**Key principle:** Client and solo share the same data model — `coach_id` is nullable.
A solo user = a client with no coach attached. One profile type, not two systems.

## Permissions model

| User | Can see | Can write |
|------|---------|-----------|
| Coach | All their clients, all data | Everything for their clients |
| Client | Own profile only | Own weight logs, session logs, feedback, personal calendar events |
| Solo | Own profile only | Everything on own profile |

Coach has **read access** to client-added calendar events (holidays, competitions affect programming).
Coach cannot edit or delete client-created events.

## Invite flow
1. PT creates client record in CoachApp
2. PT clicks "Send invite" → Supabase `inviteUserByEmail()` sends magic link
3. Invite URL carries PT's ID as metadata
4. Client clicks link → lands on signup page → account created → auto-linked to PT
5. v1: Supabase default email branding. v2: customise with PT's name/logo.

## Signup flow
Two explicit paths on the signup page:
- "I'm a coach or PT" → role: `coach`
- "I'm training for myself" → role: `solo`
Client accounts are created only via PT invite — no self-signup as `client`.

## Dashboard (PT — Monday morning view)
- Upcoming sessions this week (per client)
- Recent client updates (weights logged, sessions completed)
- Any feedback submitted by clients
- Flagged items (missed sessions, goal deadlines approaching)

## Client dashboard
- Own upcoming sessions (PT-assigned + personal events)
- Personal calendar (PT sessions shown, personal events added by client)
- Own weight / performance progress
- Goals set by PT

## Calendar
- PT assigns structured sessions to specific dates
- Client adds personal events: competitions, holidays, gym sessions, other
- Both event types visible in one calendar view — visually distinct (PT-assigned vs client-created)
- PT has read-only access to client-created events

## Build order

1. **Schema updates** — add `client` role to profiles, make `coach_id` nullable, add `events` table
2. **Weight tracking UI** — `weight_logs` table exists; build the UI
3. **Invite system** — PT invites clients via email; auto-link on signup
4. **Client dashboard + calendar**
5. **Performance / PB tracking**
6. **Programs** (phases, weeks, day slots)
7. **Solo user flow** (signup path + stripped-down dashboard)
8. **Macros, progress photos, invoices, comms log** — lower priority

## What we are NOT building in v1
- Client self-signup (invite only)
- Native mobile app (PWA first, Capacitor later)
- Custom invite email branding (v2)
- AI features
- Marketplace / PT discovery

_Last updated: 2026-06-19_
