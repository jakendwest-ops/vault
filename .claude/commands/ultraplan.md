---
description: Deep planning session — the local stand-in for Claude Code's cloud /ultraplan. Withdraw vault context, plan at depth scaled to stakes, adversarially critique, then present a reviewed plan you approve before anything executes.
---

# /ultraplan — deep planning session

The local equivalent of Claude Code's cloud `/ultraplan`. The cloud command drafts a plan in a remote session, has you review it in the browser, then executes remotely or hands the steps back to the terminal. This harness has no cloud round-trip, so the mapping is faithful but local:

- **draft at depth** → withdraw, then plan with effort scaled to the stakes (§1–§5)
- **review in browser** → present the plan in chat *and* file it to disk as the reviewable artifact (§6)
- **execute remotely / back to terminal** → on the owner's word, execute the steps here or hand back the numbered list (§7)

Nothing outward, irreversible, or destructive-beyond-the-vault happens before approval — duty 4 holds without exception. This is planning, not doing.

The task: **$ARGUMENTS**
> If that is empty, ask for the objective in one line before going further — do not invent one.

## 1. Withdraw first

Before deriving anything, check what the vault already knows (compounding contract §1). Infer the project from the task; read its `STATUS.md` and `CRITICAL.md`, the `LOG.md` tail if continuity matters, and any prior `analysis/` on this question. Run a `qmd` search across the relevant collection. A loaded next-action is historical until verified against disk. Note what you found so the plan builds on memory instead of re-deriving it.

## 2. Frame the problem

State, in a few lines: the objective, the real constraints (residency, deadlines, dependencies, the owner's north star), what "done" looks like, and your read of the **stakes** — reversible/low, or outward/irreversible/multi-domain/high. When stakes are ambiguous, resolve upward (CLAUDE.md). The stakes read sets the depth in §3.

## 3. Plan at depth — scaled to the stakes (adaptive)

- **Low stakes · single-domain · reversible** → one deep pass yourself (or a single `Plan` subagent). No fan-out; depth without ceremony.
- **High stakes · multi-domain · outward · hard to reverse** → fan out **2–4 architect subagents in parallel** (`Plan` and/or `general-purpose`), each given a genuinely *different* angle — e.g. fastest-path, risk-first, cheapest-to-reverse, owner-experience-first — each armed with the relevant `System/craft/` file and a clear contract (objective, purpose, done-when, honest return states). Diverse lenses beat N identical planners.

Each architect returns a candidate plan with its key risks and assumptions named.

## 4. Critique

Send the leading plan (or the full set of candidates) to the standing `critic` agent for fresh-context adversarial review: where does it break, what did it assume without basis, what's the failure mode the owner would regret, what's missing. For a single-pass plan, still run `critic` — the whole point of "ultra" is that the plan is stress-tested before the owner sees it.

## 5. Synthesize

Resolve into **one** plan, grafting the best ideas from the runners-up. Every step carries a **verifiable done-when** — the stated check, not the intention (duty 2). Mark inference as `INFERRED` with its basis; mark unknowns unknown. Sequence so the cheapest-to-reverse and the riskiest-to-discover come first. Name the assumptions the whole plan rests on.

## 6. Present & file — the "review" step

Deposit before you stop (compounding contract §2) — the plan is valuable output and must not be stranded in chat:

- **Project-scoped** → write to that project's `analysis/` (e.g. `analysis/YYYY-MM-DD-<slug>-plan.md`).
- **Cross-cutting or OS-level** → write to `Vault/desk/plans/YYYY-MM-DD-<slug>.md` (create the folder the day it's first needed).

Then present the plan in chat: the frame, the numbered steps with their done-whens, the named risks and assumptions, and what you'd watch. This is the artifact the owner "reviews" — the local stand-in for the browser review.

## 7. Stop for approval

**Do not execute until the owner says go.** On approval, offer two paths, matching the cloud command's fork:

- **Execute here** — run the steps in order, verifying each done-when against disk before moving on, depositing results as you go.
- **Hand back** — deliver the numbered steps for the owner to run themselves.

If the consequential judgments in this plan are the kind reality will later grade (a recommendation, a forecast, a call), capture them as predictions with a `verify_by` date per the compounding contract — that is owed whether or not the plan is executed now.
