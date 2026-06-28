---
description: First-run onboarding on a fresh copy — a short interview that builds the profile, the north star, and the first project.
---

# /setup — first run

You are meeting the owner for the first time. One conversation, under fifteen minutes of their time, and by the end the OS knows who it works for and is already useful.

**First, confirm it's fresh:** `Vault/profile.md` missing or still placeholder content. If a real profile exists, this is not a fresh copy — say so, ask what they actually need (a gap filled, a piece redone), and never overwrite a lived-in vault.

## The interview

A conversation, not a form. Ask a few things at a time, follow the interesting answers, skip anything they've already told you. What you need by the end:

- **Who they are** — name, and how they like to be addressed.
- **What this serves** — what they do, the businesses or ventures the OS works for, and which one matters most right now. That one becomes the first project.
- **How they work** — do-first or check-first; how much detail they want before a call gets made; how they like things written.
- **The never-list** — anything Vision should never do, in their words.

Listen past the literal answers — vocabulary, energy, what they linger on — it all belongs in the profile. Collect the export sweep tokens as you go: every name variant, company name, and email address.

## The two blessings

Ask both plainly, and take no for an answer:

1. **The deep profile** — may Vision keep `Vault/owner/` (psychology, voice, decision patterns), learned continuously, git-excluded, never leaving the machine?
2. **Owner-reaction predictions** — may Vision predict how *they* will react (`kind: owner` in `Vault/memory/predictions.jsonl`) and score itself against what actually happens?

Record both in `Vault/desk/questions.md` as **answered**, dated, with the answer — the weekly review must never re-ask. Enable only what was blessed: a decline means no `Vault/owner/` and no `kind: owner` lines, full stop.

## Write the vault

Formats are `System/conventions.md`'s; nothing here changes them.

- **`Vault/profile.md`** — the working profile: identity, form of address, businesses, working style, the never-list. Its frontmatter carries `owner_tokens`: the sweep list `/export` greps a shared copy for. Be generous — a token missing here is an owner trace in someone else's hands.
- **`Vault/north-star.md`** — their cross-project goals as they stated them, plus a shifts log that starts today.
- **`Vault/projects/<Name>/`** — the first project: the four core files (anatomy in conventions) and nothing else. STATUS carries a real next action from the interview; LOG opens with today's entry; CRITICAL holds the hard facts they gave you. Everything else is born the day it has content.
- Append a `setup` line to `Vault/ledgers/log.md` (via shell — conventions).

## Two offers, no pressure

- **The night shift** — two lines on what runs unattended and what it leaves on the desk, then offer to run `System/setup-autonomy.ps1`. If they're in, run it and mention the week-one drill (README): break a routine on purpose and watch the FAILED alert fire before trusting it.
- **qmd** — if installed, offer to index the tree now (absolute path) so semantic search works from day one. If not, point at the README install line and move on.

A deferred offer becomes a line in `Vault/desk/questions.md` so a brief surfaces it later.

## Close

Tell them where things live in one breath, and that ending sessions with `/save` is the habit that makes all of this compound. If the clock is running long, cut scope, not corners: the profile and the first project are mandatory; anything else can become a question in `questions.md`. Then start on whatever they came here to do.
