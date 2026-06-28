---
description: The morning picture — night-shift health first, every project's pulse, the deadline horizon, the desk, overdue predictions
---

# /brief — the morning picture

Build the owner's morning picture. Same shape every time, tight enough to read in thirty seconds, ordered so nothing bad can hide below the fold. This prompt is also the morning routine's core — it runs unattended at dawn and interactively whenever the owner asks.

## Night shift first

Before anything else, audit `Vault/ledgers/runs.jsonl` since the last brief (the newest file in `Vault/desk/briefs/` marks the window; no prior brief means audit the whole ledger). Match every run's open record to its close record — a run without a close record is presumed failed, whatever else the ledger says. Night-shift health opens the brief: one quiet line when all is well, the failure named loudly at the top when it isn't. A buried failure defeats the entire watchdog chain — never summarize one into the middle.

## Then assemble

**Projects.** Glob `Vault/projects/*/STATUS.md`, skipping `_archive/`. One line each: next action, blockers, anything needing the owner. Flag any STATUS whose `_Last updated_` stamp disagrees with the file's mtime — edited without restamping, or stale for weeks; either way the live state is lying, and the brief says so. **A confidential project** (listed in the `.gitignore` Confidential block) gets a content-free line in any brief written to disk — name + "see project STATUS", nothing more: briefs are committed, and confidential detail must never escape into committed files through a summary. Interactively, in chat, full detail is fine.

**Deadline horizon.** Grep `due:` and `verify_by` across the vault (marker conventions: `System/conventions.md`). List everything within 14 days, soonest first. Anything overdue goes first and loud — its own line, not folded into the list.

**The desk.** What's waiting on the owner: drafts in `Vault/desk/outbox/` awaiting approval (surface the five oldest with ages and the total count — never the whole pile; a long list trains the reflex-approve the outbox exists to prevent), open items in `Vault/desk/questions.md`, proposals in `Vault/desk/proposals.md` still awaiting a verdict.

**Predictions.** The count of overdue predictions in `Vault/memory/predictions.jsonl` — `verify_by` past, no outcome. One number; scoring them is `/review`'s job.

## Delivery

- **As the morning routine:** write the brief to `Vault/desk/briefs/YYYY-MM-DD.md` and keep it tight — a page the owner reads with coffee, not a report. Never overwrite an existing day's brief: if the file exists, write `YYYY-MM-DD-2.md` instead.
- **Interactive:** deliver it in chat, then offer to file it to the same path.
