---
id: 2026-08-29-two-orphaned-functions-and-a-stale-dom-fallback
status: open
priority: low
reported: 2026-08-29
status_detail: "addExtraStrengthSet (app-runner.js:2076) has zero references repo-wide - its onclick died with the wizard, superseded by addTableRow. saveTemplateToLibrary's fallback getElementById('sd-save-library') targets an id that exists nowhere; the sole caller always passes 'this', so it is harmless, but it documents a button openSessionDetail explicitly says it no longer renders."
---

# Two orphans left by the wizard deletion

Found by the weekly full-file review (Agent C). Both are cleanup, filed so they are not rediscovered.

1. **`addExtraStrengthSet` (js/app-runner.js:2076-2080)** — **zero** references repo-wide. Its
   `onclick` lived in the deleted wizard; the only remaining copy is in a worktree. Superseded by
   `addTableRow`.
2. **`saveTemplateToLibrary`'s fallback (js/app-workouts.js:2820)** —
   `document.getElementById('sd-save-library')` targets an id that **exists nowhere**. The sole caller
   (`app-programs.js:2124`) always passes `this`, so it is harmless — but the fallback documents a button
   that `openSessionDetail:486-487` explicitly says it no longer renders.

These are the residue of the wizard deletion, the same source as
`2026-08-12-dead-code-post-wizard-deletion-runner` and
`2026-08-17-dead-code-and-lost-affordances-after-the-wizard-deletion`. **Worth consolidating with those
two rather than fixing in isolation** — three rows now describe one cleanup.

**Closes when:** both are removed (or the fallback's id restored if the button returns), and a grep for
each name returns only the definition site or nothing.
