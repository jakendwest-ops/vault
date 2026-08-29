---
id: 2026-08-29-unescaped-db-strings-reach-innerhtml-at-3-sites
status: open
priority: medium
reported: 2026-08-29
status_detail: "nextEx.name is escaped at app-runner.js:803 and :814 but raw at :607/:613 and :1685/:1703 - four renders of one field, two safe, two not. programName is raw at app-workouts.js:3071 while its sibling six lines below is escaped. Self-XSS today (names are coach-authored) but the 6th instance of a sink class CRITICAL.md tracks."
---

# Unescaped DB strings reach `innerHTML` at 3 sites, each beside an escaped sibling

Found by the weekly full-file review (Agent A). **Verified by grepping each cited line for
`escapeHtml`** — the asymmetry is exact:

| Line | Field | `escapeHtml`? |
|---|---|---|
| `app-runner.js:607` -> `:613` | `nextEx.name` (inline rest bar) | **no** |
| `app-runner.js:1685` -> `:1703` | `nextEx.name` (floating rest overlay) | **no** |
| `app-runner.js:803` | `nextEx.name` | yes |
| `app-runner.js:814` | `_runner.exercises[...].name` | yes |
| `app-workouts.js:3071` | `programs.name` | **no** |
| `app-workouts.js:3077` | `workout_templates.name` (6 lines below, same literal) | yes |

**Four renders of one field; two escaped, two not.** This is drift, not an unknown gap — the correct call
already exists 200 lines away in the same file.

**Where it fires:** both `nextEx.name` sinks render during **rest**, which is unavoidable in a normal
session. An exercise named `<img src=x onerror=...>` sitting after the current one executes in the
session of whoever drives the runner — including a **coach**, who reaches it via "▶ Start workout" on the
client tab, with the Supabase token in localStorage.

**Honest severity: self-XSS today.** `exercise_name` and `programs.name` are coach-authored; a client
cannot write them (`tests/template-exercise-write-rls-2026-08-10.spec.js:138` proves a client cannot
write their own plan clone's exercises). So there is no cross-tenant path today. It is filed because it is
the **6th instance** of the sink class `CRITICAL.md` tracks, and because a fix is one call per site.

**Fix:** `escapeHtml(nextEx.name)` at `:607` and `:1685`; `escapeHtml(programName)` at `:3071`.

**Closes when:** all three sites are escaped AND `scripts/check-escaping.mjs` is shown to catch this
syntactic form — see [[feedback_reports_success_doing_nothing]], whose merged section records that this
same checker ran clean over nine live sites because only one syntax shape was ever considered. Plant an
instance in the exact form used here (`'Next: ' + ex.name` assigned to a variable, then interpolated) and
confirm the checker goes RED before trusting a clean run.
