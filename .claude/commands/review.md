---
description: The weekly learning ritual — score due predictions, write the scorecard into the brief, ask one profile question, file improvement proposals, trend-check the night shift.
---

# /review — the weekly learning ritual

This is where the OS faces its own score. The point is not a good-looking scorecard — it is a true one. This same ritual is the core of the weekly routine (`System/routines/weekly.md`), run unattended on its schedule; the owner typing `/review` runs it with connectors in reach.

Formats for every record touched here: `System/conventions.md`.

## 1. Score due predictions

Read `Vault/memory/predictions.jsonl` — and `Vault/owner/predictions.jsonl` if the reaction-prediction blessing is active (`kind: owner` lines live there, inside the never-committed residency) — by file and grep. For every open prediction past its `verify_by`:

- Outcome determinable from the vault or disk → record it, with a score note naming the gap between claim and reality.
- Outcome needs live external data → mark it `DUE-NEEDS-CONNECTOR`. That is honest debt, never a silent skip: it appears in the scorecard, and the next interactive session pulls the outcome. If you *are* that interactive session, clear the backlog now, before anything else.
- A scored miss writes a `kind: calibration` lesson to `Vault/memory/lessons.jsonl` — what was predicted, what happened, what correction applies next time. The do-not-capture filter still holds: bank the measured correction, not the embarrassment.

## 2. The scorecard

Write it into the weekly brief — `Vault/desk/briefs/YYYY-MM-DD.md`, appended as a section if the morning brief already exists. Every number freshly counted from the files, never carried from last week:

- **Beliefs** — total · new this week · stale (unconfirmed 60+ days)
- **Predictions** — scored · open · overdue (including `DUE-NEEDS-CONNECTOR` debt)
- **Calibration** — by kind and by project: where confidence ran hot, where it ran cold
- **Lessons banked** this week · **plays promoted** to playbook pages
- **Outbox** — pending count · median age-to-decision · approve/decline ratio this week. A growing median age or a near-100% reflex-approve rate is the rubber-stamp warning the outbox design depends on catching — flag it.
- **Memory vs. re-derived** — the honesty check: did this week's sessions answer from what the vault already knew, or re-derive it? Name the instances either way.

Close with this clause, verbatim: **"if these numbers don't climb, the OS isn't learning — and it says so."** If they aren't climbing, say so right there, plainly, with your best reading of why.

## 3. One profile question

Append ONE owner-profile gap question to `Vault/desk/questions.md` — the gap whose answer would most change how you work for the owner. If last week's is still unanswered, skip this week; questions don't pile up.

## 4. Self-improvement proposals

From observed friction and the run-ledger trends, file proposals with evidence into `Vault/desk/proposals.md`. **First read the Declined section — a declined idea is never re-proposed.** Nothing worth proposing is a fine outcome; a manufactured proposal is not.

## 5. Night-shift trends

Read `Vault/ledgers/runs.jsonl` over the trailing weeks and judge trends, not days: "nightly failed 3 of 7" matters; one bad night is noise. A degrading routine becomes a proposal or a flag in the brief — pick by severity.

## 6. Light checkup

Run a light `/checkup` pass. Anything material lands in the brief.
