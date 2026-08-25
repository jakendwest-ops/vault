# Not the project brief — read the real one

**The CoachApp project brief lives at `C:\Users\jaken\OneDrive\coachapp\CLAUDE.md`.** That file is
auto-loaded every session and is the one that is kept current. This file is not a mirror of it and
should not be treated as one.

## Why this file still exists

Until 2026-08-25 it claimed to be a mirror and then gave instructions for a tool called `graphify` —
telling the reader to run `graphify query "<question>"` for codebase questions and `graphify update .`
after changing code.

**There is no `graphify` on this machine and no `graphify-out/` in the project.** Verified 2026-08-25:
`which graphify` finds nothing, the directory does not exist, and the string appears nowhere in the
repo. The content had been untouched since 2026-07-02.

That made it the most dangerous artefact the OS v3 inventory found — not stale, *wrong*, and sitting
exactly where a future session would reasonably look for the project brief. Stale content misleads;
this would have sent a session off to run commands that do not exist.

**It was flagged for deletion, and the deletion was refused by the permission classifier.** Rather
than route around a denial — a standing rule here, and its own ledger row — the content was replaced
so it can no longer mislead anyone. **Deleting the file outright is still Jake's call.**

## The lesson, kept because it generalises

This is what a written update obligation with no writer decays into. The old first line promised
*"tell Claude to port changes across"* — a promise nothing checked for 54 days. os-lint's
`doc-obligations` check now covers documents that declare they must be kept current, which is RULE 0
applied to documents rather than only to incidents.

See `STATUS.md`, `LOG.md`, `CRITICAL.md` and `roadmap.md` in this directory for the real state.
