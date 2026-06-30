# CoachApp — Data Model

_Living design doc. Update whenever the schema changes — keep in sync with [[project-coachapp-architecture]] and CRITICAL.md. Mermaid renders directly on GitHub and most markdown viewers — no external tool needed._

---

## Core entity relationships

```mermaid
erDiagram
    CLIENTS ||--o{ WEIGHT_LOGS : logs
    CLIENTS ||--o{ PERFORMANCE_LOGS : logs
    CLIENTS ||--o{ WORKOUT_LOGS : logs
    CLIENTS ||--o{ GOALS : has
    CLIENTS ||--o{ EVENTS : has
    CLIENTS ||--o{ CLIENT_CHECK_INS : submits
    CLIENTS ||--o{ CLIENT_1RMS : has
    CLIENTS ||--o{ CLIENT_PROGRAMS : assigned

    PROGRAMS ||--o{ PROGRAM_PHASES : has
    PROGRAMS ||--o{ CLIENT_PROGRAMS : "assigned as"
    PROGRAM_PHASES ||--o{ CLIENT_PROGRAM_WORKOUTS : schedules

    CLIENT_PROGRAMS ||--o{ CLIENT_PROGRAM_WORKOUTS : contains
    CLIENT_PROGRAM_WORKOUTS }o--|| WORKOUT_TEMPLATES : "runs (client copy)"

    WORKOUT_TEMPLATES ||--o{ WORKOUT_TEMPLATE_EXERCISES : contains
    WORKOUT_TEMPLATES ||--o{ WORKOUT_LOGS : "logged from"

    WORKOUT_LOGS ||--o{ WORKOUT_LOG_EXERCISES : contains
    WORKOUT_LOG_EXERCISES ||--o{ WORKOUT_LOG_SETS : contains

    GOALS ||--o{ GOAL_MILESTONES : has
    GOALS ||--o{ GOAL_CHECK_INS : has

    CLIENTS {
        uuid id PK
        uuid user_id FK "unique — one record per user"
        uuid coach_id FK "null = solo/personal account"
    }
    PROGRAMS {
        uuid id PK
        uuid coach_id FK
    }
    WORKOUT_TEMPLATES {
        uuid id PK
        uuid program_id FK "set = master template"
        uuid client_id FK "set = client plan clone"
    }
```

---

## The two-level editing model (the part that matters most)

This is the single highest-risk area in the schema — getting it wrong means one client's edit silently changes another client's plan, or vice versa. See [[project-coachapp-architecture]] for the full rule.

```mermaid
flowchart TD
    A[PT builds a Program] -->|workout_templates: program_id SET, client_id NULL| B[Master Template]
    B -->|Assign to Client X| C["Clone row-by-row\n(client_programs + client_program_workouts)"]
    C -->|workout_templates: program_id NULL, client_id SET| D[Client X's Plan Copy]
    B -->|Assign to Client Y| E["Clone row-by-row"]
    E -->|workout_templates: program_id NULL, client_id SET| F[Client Y's Plan Copy]

    D -->|PT edits Client X's plan| D
    D -.->|"NEVER affects"| F
    D -.->|"NEVER affects"| B

    B -->|PT edits the master Program| G["Future assigns only —\nexisting client copies untouched"]

    style B fill:#6366f1,color:#fff
    style D fill:#22c55e,color:#fff
    style F fill:#22c55e,color:#fff
    style G fill:#f59e0b,color:#fff
```

**Rule:** `program_id` set + `client_id` null = master. `program_id` null + `client_id` set = personal copy. Never both set, never both null for a real template (standalone templates are both null). No auto-renaming — template names only change manually, which is what lets "apply to all sessions named X" work reliably.

---

## Account types / roles

```mermaid
flowchart LR
    U[auth.users] --> P[profiles]
    P -->|role: coach| PT[PT account]
    PT --> CL1["clients.coach_id = PT's id\n(one row per coached client)"]
    PT -.->|"if PT also has own client record\nwith coach_id = null"| SOLO[Personal / Solo view]

    P -->|role: client| CLV[Client account]
    CLV --> CR["clients row\nuser_id = auth.uid()\ncoach_id = assigned PT"]

    SOLO --> SR["Same clients row,\ncoach_id = null\n(self-coached)"]

    style PT fill:#6366f1,color:#fff
    style CLV fill:#22c55e,color:#fff
    style SOLO fill:#f59e0b,color:#fff
```

**Key facts:**
- `clients.user_id` has a unique constraint — exactly one client record per user, ever.
- Solo/Personal is **not** a separate record — it's the same client record with `coach_id` nulled.
- `window._masterAccount` = true when the logged-in coach also has any client record (coached or personal).
- RLS anchor for solo: `client_id in (select id from clients where user_id = auth.uid() and coach_id is null)`.

---

## Workout logging flow (runner)

```mermaid
flowchart TD
    A[Client/PT starts session\nfrom Workouts page] --> B[workout_logs row created]
    B --> C[Runner: set-by-set logging]
    C --> D[workout_log_exercises\none row per exercise]
    D --> E[workout_log_sets\none row per set]
    C -->|"%1RM set type"| F[client_1rms\nlooked up for target weight]
    E --> G[Session summary / finish screen]
    G -->|optional| H[Save estimated 1RM\nfrom today's sets — planned]

    style F fill:#f59e0b,color:#fff
    style H fill:#94a3b8,color:#fff,stroke-dasharray: 5 5
```

The dashed box is the 1RM system work planned for this session — not built yet.

---

## How to keep this in sync

- Any schema change (new table, new FK, renamed column) → update this file in the same session.
- This file lives in the Vault, not the repo — it's the design reference, not enforced by code. The source of truth for actual constraints is Supabase itself (`information_schema`) and `CRITICAL.md`.
- Don't duplicate this in Notion or any other tool — one file, versioned with the rest of the Vault.
