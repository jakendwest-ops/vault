---
description: Ingest a source or batch — classify up, convert on-device, merge into existing pages, stamp provenance, log it.
---

# /ingest

**Source:** $ARGUMENTS — a file path, a URL, a folder, or a mix. Nothing passed → ask what to ingest.

The pipeline and its order are fixed in `System/conventions.md` § Ingestion. This file adds the operational detail; it is not a second copy of the rules.

**Classify before any tool reads the content.** Judge sensitivity from what the source *is* — origin, filename, which project it belongs to — because that call decides which tools are allowed to touch it.

**Convert on this machine.**
- Documents and URLs → markitdown: the MCP tool, or the `markitdown` CLI if the server isn't wired.
- Audio → local Whisper. Scans and images → local OCR.
- A sensitive source that can't be read on-device **stops and flags**: report what couldn't be read, why, and what would unblock it (install Whisper, rescan at higher resolution, owner types the key figures). Never fall through to a cloud transcription, OCR, or vision service. Continue with the rest of the batch.

**Merge before you create.** qmd-search first — lex on the source's distinctive terms, vec on what it's about — to find the pages that already own this ground. Those are the default destination: fold the new material in, append changed facts to their timelines, supersede rather than sit a duplicate beside the original. A new page is for the genuinely novel only — content no existing page is the home of. Destinations and page shape: conventions.

**Stamp provenance.** Every page this ingest created or merged into gets `provenance: untrusted-derived` in its frontmatter — what the stamp means and why it travels with the page: conventions § Page anatomy.

**Report once.** Single source or batch, end with one summary: each source → what it merged into or what it created, plus anything that stopped-and-flagged. The owner should see the whole batch's fate at a glance.

**Close the ledger.** Append one `ingest` line to `Vault/ledgers/log.md` via shell — editor writes there are denied by design:

```powershell
Add-Content -Path 'Vault/ledgers/log.md' -Value '## [YYYY-MM-DD] ingest | <project or vault area> | <sources -> destinations, one line>'
```

Real date, real summary; line format: conventions § Ledgers.
