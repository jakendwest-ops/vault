---
description: Produce the born-clean shareable copy — framework only, no Vault/, no git history, certified free of owner traces
argument-hint: <destination path>
---

# /export — the born-clean copy

Destination: `$ARGUMENTS` — if empty, ask the owner where the copy should land (a path outside this repo).

The product is a fresh Vision OS a stranger could clone and trust: the framework, nothing of the owner. Every check below must pass before you call it shipped; the sweep in particular is never waved through.

**Copy the framework.** robocopy this repo to the destination, recursive, excluding: `Vault\` wholesale, `.git`, `.obsidian`, `.claude\settings.local.json`, any `*.local.*` file, and `System\design\` (the genesis record is this instance's build history — owner-specific by nature, not framework). (robocopy exit codes 0–7 are success; 8+ is failure.)

**Templatise the boundary file.** The copied `.gitignore`'s Confidential block names the owner's confidential folders — that is the one framework file that is owner data by design. In the copy, replace the block's entries with the empty commented placeholder (`# (confidential folders are appended here by flagging a project confidential)`), keeping every other section verbatim.

**Templatise the launch config.** `.claude/launch.json` carries the owner's machine-specific dev-server configurations (absolute paths, usernames, project names) and is git-ignored as machine-local. In the copy, replace its `configurations` array with an empty `[]`, leaving the file as a format scaffold a newcomer can fill in. (If the file is absent in source, ship a blank one with `configurations: []`.)

**Sweep the copy for owner traces.** Read `owner_tokens` from the frontmatter of `Vault/profile.md` — in this repo, the source; the copy has no Vault. If the list is missing or empty, stop: an export cannot be certified clean without its sweep dictionary. Search the copy for every token — case-insensitive, fixed-string, every file of every type (prompts, scripts, configs, docs alike): `rg -i -F --hidden --no-ignore`, so the copy's own `.gitignore` doesn't blind the search.

**ANY hit is a hard fail.** Report each token with the files it appears in, then stop. The fix happens at source — edit or generalise the framework file carrying the trace *here*, then re-run `/export` from scratch. Never patch the copy by hand, never ship with a hit.

**Verify before asserting done.** In the copy: no `.git` directory anywhere (check it — don't trust the exclusion); `CLAUDE.md`, `README.md`, `System/`, and `.claude/` present and intact.

**Report.** The destination path, what was excluded, the sweep result (tokens checked, zero hits), and one closing line: a fresh copy has no `Vault/`, so it boots straight into `/setup` on first hello.
