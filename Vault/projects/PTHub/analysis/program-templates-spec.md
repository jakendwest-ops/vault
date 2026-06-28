# Program Templates — Feature Spec

_Filed 2026-06-15. Not yet built — parked for a future session._

## What it is

Reusable multi-week training blocks that sit above individual workout templates. A program is assigned to a client with a configurable schedule and duration.

## Core concepts

- **Program** — a named block (e.g. "12-Week Hypertrophy", "5/3/1 Block") containing an ordered list of **phases** or **weeks**
- **Week** — contains session slots mapped to days (Mon, Wed, Fri etc.) each pointing to a workout template
- **Assignment** — a program assigned to a client with a start date; generates recurring calendar events automatically
- **Progress tracking** — My Progress tab should be filterable by assigned program so the PT can see progression within a block

## Data shape (proposed)

```js
// global programs (not per-client — shareable across clients)
programs: [
  {
    id, name, description,
    weeks: [
      {
        weekNum: 1,
        sessions: [
          { day: 'mon', templateId: 123, label: 'Push' },
          { day: 'wed', templateId: 124, label: 'Pull' },
          { day: 'fri', templateId: 125, label: 'Legs' },
        ]
      },
      // ...repeat for N weeks
    ]
  }
]

// per-client assignment
c.programAssignments: [
  { id, programId, programName, startDate, weeks: 12, active: true }
]
```

## UX notes

- Programs panel: either a global tab on the dashboard, or a fourth tab inside Workout Builder ("Programs")
- Assignment: on client profile, a "Assign Program" button that picks from the program library + sets start date
- Calendar integration: assigning a program auto-generates session events for the duration
- My Progress: add a "Filter by program" dropdown so progression charts reflect a specific training block

## Dependencies

- Workout templates must exist first (already built)
- Calendar events panel (already built) — program assignment writes to it
- My Progress tab (just built) — needs program filter layer

## Effort estimate

Medium-large. UI is the bulk of the work — the data model is straightforward.
