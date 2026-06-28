# Vision OS 3.0 — Fable Edition

A personal knowledge operating system that runs on [Claude Code](https://claude.com/claude-code) and lives as a markdown wiki you browse in [Obsidian](https://obsidian.md). Vision — the assistant this OS shapes — maintains it: files what you learn, remembers what you decide, predicts and then scores itself against reality, and works a night shift so mornings start with a brief instead of a backlog.

Built for a capable model under **maximum trust, minimal rails**: four hard safety rails live in configuration; everything else is judgment. The system's worth is measured one way — *does it compound?* It answers from memory instead of re-deriving, its predictions get sharper, and it says so plainly when they don't.

**New here? Open [`User Guide.html`](User%20Guide.html) in a browser** — the full human's guide: every command, the night shift, the outbox, the desk, troubleshooting. This README is the short orientation.

## The one boundary

Everything that is *yours* — profile, projects, memory, ledgers — lives under `Vault/`. Everything else is shareable framework.

- **Backup:** commit the repo to a *private* remote. Sensitive folders (health, client data, the deep profile) are excluded by `.gitignore` and never leave the machine — see the "Confidential" block at the top of that file; it is the single record of what's excluded.
- **Sharing/export:** `/export` produces a copy with `Vault/` removed and zero owner traces — born clean, no git history.

## Day one

1. Install: Claude Code + pwsh 7; optionally Obsidian (open the repo as a vault), [qmd](https://github.com/tobi/qmd) for semantic search, `uvx markitdown-mcp` for ingestion, and [gitleaks](https://github.com/gitleaks/gitleaks) if you want the pre-commit secret scan to actually run (`.gitleaks.toml` configures it; without the binary it's an owner-run manual gate, and `/checkup` says so rather than pretending it ran).
2. Open Claude Code in this folder. On a fresh copy, say hello — Vision runs `/setup`: a short interview that builds your profile and first project.
3. Work. Vision files as you go; end sessions with `/save`.
4. When ready for the night shift: run `System/setup-autonomy.ps1` (registers the three scheduled routines), then break one on purpose in week one — you should see the FAILED alert. Trust it only after you've watched it fail loudly.

## The commands

| Command | What it does |
|---|---|
| `/save` | Deposit the session: STATUS rewritten, LOG appended, predictions captured. End every working session with it. |
| `/brief` | The morning picture: night-shift health, every project's next action, deadline horizon, outbox awaiting your eye, open questions. |
| `/review` | The weekly learning ritual: score due predictions, the honesty scorecard, one profile question, improvement proposals. |
| `/checkup` | Health sweep: boundary intact, ledgers append-only (git-prefix check), drift hunt, stale-belief flags. |
| `/ingest <source>` | File a URL/document/audio: on-device conversion, rewrite-merge into existing pages, provenance stamped. |
| `/setup` | First-run interview on a fresh copy. |
| `/export` | The born-clean shareable copy. |

Everything else — search, capture, recall, delegation, synthesis — Vision just does; the kernel makes them duties, not commands.

## The night shift

Three routines run unattended via Task Scheduler → `System/run.ps1` → `claude -p` under no-egress profiles: nightly and morning get `System/headless.json` (no MCP, no web, no network commands), the Sunday review alone gets `System/headless-weekly.json` — identical except for enumerated **read-only** connector tools used to score predictions. No unattended session can send, publish, or transact, ever:

- **Nightly** — inbox triage, cross-project fact reconciliation, deadline flagging, drafting outward actions into `Vault/desk/outbox/` for your one-look approval, backup push.
- **Morning** — audits the night's runs first, then writes the dated brief to `Vault/desk/briefs/`.
- **Weekly** — scores due predictions, writes the learning scorecard ("if these numbers don't climb, the OS isn't learning — and it says so"), asks one profile question, files improvement proposals.

Every run writes an affirmative close record (`OK | OK_EMPTY | PARTIAL | FAILED`) to `Vault/ledgers/runs.jsonl`. A run that didn't report is presumed failed, and the runner itself writes the FAILED record and a desk alert — silence is never mistaken for success.

## How it learns

Consequential judgments become predictions with `verify_by` dates (`Vault/memory/predictions.jsonl`). Reality grades them; the gaps become calibration lessons that load into future work. Beliefs and lessons accumulate in append-only JSONL you can read with any text editor. The deep owner profile (`Vault/owner/`, never committed) learns your voice from the diff between Vision's drafts and what you actually ship.

Vision also proposes its own upgrades — with evidence — in `Vault/desk/proposals.md`. You approve or decline in chat; declined ideas stay on the record and are never pitched twice.

## The lineage

3.0 is a ground-up rebuild of Vision OS 1.0 (the experiment: 13 laws, 23 agents, 43 commands, 11 hooks) and 2.0 (the consolidation: 12 laws, 7 agents, 24 commands, 7 hooks), built for a model that no longer needs governing. What survived is what the evidence said earned its keep; the full reasoning, with the evidence trail, is `System/design/2026-06-10-genesis.md`.
