---
description: The health sweep — residency boundary, ledger append-only proof, drift hunt, night-shift audit. Fixes the mechanical, proposes the structural.
---

# /checkup — the health sweep

Run the sweep below, fix what is mechanical and in-vault as you go, and finish with one short report. The weekly routine runs this light: same boundary and ledger checks in full, smaller drift sample. In a headless run a check can be refused by permission — the no-egress profile is deliberately narrow; a refused check is named in the report as **skipped, with the reason** — never silently omitted, never improvised around.

## The boundary

The Confidential block of `.gitignore` is the list — read it there, never keep a copy here. For each path in it:

- it must show as ignored in `git status --ignored`
- `git ls-files <path>` must return nothing — zero tracked files under a confidential folder, ever.

Then `git ls-files -ci --exclude-standard` for the same breach from the other direction: tracked files that match any ignore rule. And `gitleaks protect --staged` if gitleaks is installed — if it isn't, say so in the report rather than letting the check silently vanish.

If a `backup` remote is configured (`git remote -v`), verify the repository is actually **private** (`gh repo view <repo> --json visibility` when gh is available; otherwise flag it for the owner to eyeball). A public backup remote is a headline finding — the nightly push would be publishing the vault.

A tracked sensitive file is never a silent fix. It is already in history; surface it as the headline finding and let the owner decide what to do about history.

## Ledger integrity

For every file under `Vault/ledgers/` that exists in HEAD: the committed version must be an exact byte-prefix of the working version. Compare `git show HEAD:<path>` against the raw bytes on disk — equal is fine (nothing appended yet), committed-shorter-and-matching is fine (appends only). Anything else means history was edited or reordered — the most serious finding this command can produce. Never repair it: report it loudly with where the divergence starts and leave the working file untouched. A wholesale line-ending rewrite fails this check and deserves to — something other than an append touched the file. Files not yet in HEAD pass by default. (The append-only rule itself: `System/conventions.md`.)

## Drift hunt

Sampled and judgment-sized — enough to know whether the vault is honest, not an exhaustive crawl. Hunt at least:

- **Dangling wikilinks.** Sample `[[...]]` links across the vault and resolve them (link forms: `System/conventions.md`). A link whose target obviously moved or was renamed: fix it. A link to something that never existed: report it.
- **STATUS staleness.** Each project STATUS's `_Last updated_` stamp against the file's mtime. Material disagreement means something rewrote STATUS without restamping — correct the stamp and note it.
- **Stale beliefs.** Any line in `Vault/memory/beliefs.jsonl` whose `last_confirmed` (or `date`, if never confirmed) is more than 60 days old goes in the report for the owner's eye. Never auto-confirm — confirming a belief is a judgment about the world, not file hygiene.
- **Placeholder rot.** TBD / TODO / template boilerplate sitting in core project files. Delete what is plainly dead, fill what disk can answer, report what needs the owner.

## The night shift

Tail `Vault/ledgers/runs.jsonl` over the last week or so: `FAILED`, `PARTIAL`, and runs that simply didn't happen on schedule all go in the report — a missed run counts as a failure.

Confirm the scheduled tasks behind `System/routines/` are still registered (`Get-ScheduledTask`) and that their actions point at **this tree's** `System/run.ps1` — a stale task running against a dead tree is the classic silent failure. The fix is `System/setup-autonomy.ps1`, but that touches the machine, not the vault: report it with that one-line fix and offer to run it; don't run it unbidden.

## Fix or propose

Mechanical and inside the vault — a dangling link with an obvious target, a stale stamp, dead boilerplate — fix it as you find it, no ceremony, and list it under Fixed. Structural — a recurring pattern, a format problem, anything that would change how the OS works — one evidence-backed entry in `Vault/desk/proposals.md`, after checking its Declined section. Anything outside the vault, and any boundary or ledger finding, is reported, never silently handled.

## The report

One short block in chat, shaped like:

```
Checkup YYYY-MM-DD
Boundary:    clean | <finding>
Ledgers:     append-only verified | <finding>
Drift:       clean | <fixed + what's left for the owner>
Night shift: <runs verdict> · <tasks verdict>
Fixed:       <one line each>
Needs owner: <items — omit the line if none>
```

A clean bill is exactly that — a few lines saying clean. Never pad a healthy system into looking interesting.
