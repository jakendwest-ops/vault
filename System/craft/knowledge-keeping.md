# Knowledge-keeping — craft

_For future Vision: load this when the vault's **shape** is the work — filing something hard to place, hunting duplicates, restructuring or migrating a corpus, auditing findability, or stewarding the memory record. Not needed for routine saves._

## Mental models

- **The four IA systems** — organisation, labelling, navigation, search. When a collection feels messy, name *which one* failed before redesigning all four. Most "the vault is a mess" complaints are one broken system, not four.
- **Findability is multi-modal.** A node should be reachable by more than one independent route — browse (hierarchy), search (query), link (association). One path is the fragile case; the modes are complements, not substitutes.
- **The controlled-vocabulary spectrum** — flat list → synonym ring → taxonomy → thesaurus → ontology. Pick the *lightest rung* that resolves the actual findability failure. Each rung up buys retrieval power at maintenance cost; most collections are over-built one rung and under-governed two.
- **Faceted beats enumerative when items live in many dimensions.** Agonising over "which folder" is the tell: the answer is orthogonal attributes combined at query time, not a deeper tree.
- **Precision vs. recall is the dial under every choice.** Broadening vocabulary, synonyms, and cross-links lifts recall and erodes precision; narrowing reverses it. "Recall collapsed" and "every search returns noise" are the same dial set wrong in opposite directions — size vocabulary depth and chunk granularity against both.
- **The records lifecycle** (create → use → maintain → dispose) is the only model that authorises destruction. It converts hoard anxiety into a defensible rule with an owner and a clock. The default remains archive (the never-lose duty); destroy only under a documented retention duty.
- **FAIR** (Findable · Accessible · Interoperable · Reusable) — the machine-actionability lens. Structure exists for retrieval agents as much as for human browsers; persistent names and rich metadata are what make a corpus a substrate rather than a swamp.

## Filing — the core decision

The sequence, every time something needs a home:

1. Does a canonical home already exist? Search first — by how the thing would be *sought*, not what it happens to be called.
2. Yes → rewrite-merge into it. The vault gets smarter, not just larger.
3. No → choose one home by how it will be sought — never by who produced it or what today's task is.
4. What metadata and links make it findable by a second independent route?
5. Does it change an existing fact? Supersede — never a quiet second copy.

It ends in a placement plus a link decision. If step 3 is genuinely agonising, that is a facet problem (above), not a filing problem.

**The second-source test:** a wikilink, an embed, or a timeline supersession is *not* a second source — only an independent restatement is. Two copies of a fact are a future contradiction with a timestamp.

## Building or auditing structure

- **Build arc:** inventory the corpus → model the user's tasks and vocabulary (a card sort *derives* the groupings) → choose the scheme → tag → wire navigation, search, and cross-links → validate (a tree test) → assign governance. Card sort builds; tree test confirms.
- **Warrant:** build categories from terms that actually occur in the corpus and in queries (literary warrant), never from an a-priori tidy taxonomy. A category nothing is filed under and nobody searches for fails warrant — it should not exist.
- **Audit:** sample real retrieval failures → mutually-exclusive or deliberately faceted? → orphans → silently diverging duplicates → unmaintained depth → vocabulary drift. The output is a merge/split/prune list, not a redesign.
- **Migration:** map old→new homes *before* moving anything; preserve provenance and redirects — a moved page with no forwarding link is a manufactured orphan. When two schemes meet (an absorbed corpus, a merger), build an explicit crosswalk: each concept mapped to its equivalent, exact vs. broader/narrower marked, the crosswalk kept as an owned artefact.

## Where senior judgment diverges

- **Lightest structure that works; escalate on evidence only.** A shallow scheme that's maintained beats a deep one that rots — maintenance cost is the dominant term, always.
- **A category that holds everything holds nothing.** Split a bucket that stops discriminating; drop a facet with one value. The value is in the distinction.
- **Scoped separation is not drift.** A near-duplicate deliberately partitioned by audience or retrieval path is *accepted* controlled redundancy — record it as intentional, cross-link both, set a review trigger. The senior call is sometimes "leave it"; the junior reflex merges every surface duplicate.
- **No taxonomy survives without an owner and a review date.** Governance, not design, is what makes a structure still real next year.
- **Structure mirrors the user's mental model** — never the org chart, never the creator's convenience. And "it's intuitive" is tested (tree test), not assumed.
- **Unknown provenance means unverifiable, not worthless.** Verify before relying; don't blindly discard.

## Failure catalogue

Over-engineered taxonomy (deep, unmaintained, collapses to noise) · build-before-validate · org-chart filing · orphans and link rot · duplicates silently diverging · tagging theatre (`DD` vs `due-diligence`; metadata with no retrieval function) · premature destruction *or* hoard-everything · the write-only archive (filed, never re-surfaced into work) · vocabulary drift · findability assumed, never tested.

## The corpus as machine substrate

- **Clean structure is AI infrastructure.** Semantic retrieval reads the corpus directly: ambiguous labels, duplicates, and orphans degrade machine recall exactly as they degrade human browsing. IA hygiene is the precondition for grounded answers.
- **Vector recall is not relational reasoning.** Similarity retrieves *near* things; multi-hop questions need explicit typed relations (the GraphRAG pattern: vector search plus a taxonomy or graph). Hold the pattern; pull the current tooling from the source-map.
- **Chunking is the new shelving.** How a page splits for retrieval is a classification decision — boundary, granularity, overlap. Bad chunking is the new mis-filing; the "For future Vision" preamble is the retrieval prefix doing this job.
- **Curation is the scarce input.** As generation gets cheap, dedup, supersession, and pruning directly raise answer quality. The librarian's judgment sits upstream of every generated answer.
- **Stale-derived honesty.** Any derived layer — an index, a topic map, a cached answer — older than its underlying file's modified-time self-labels *stale* and points at the authoritative source. Never silently serve the cache.

## Stewarding the learning record

What gets captured, where, and in what shape is stated once by the kernel and conventions. The craft is keeping the record worth reading back:

- **Design the retrieval path before the capture form.** A lesson is only as good as the future grep that finds it: write `applies_when` in the vocabulary a future task will actually carry, not the vocabulary of today's frustration. Knowledge filed but never re-entered is the write-only archive again.
- **A calibration lesson is a quantified correction, not a mood.** Direction and size, scoped: "this prediction-kind over-predicts by ~30% in this context — discount accordingly" beats "be more careful" by exactly the amount that makes it loadable.
- **Score fuzzy outcomes reason-then-grade.** State what actually happened against the claim *first*, then assign the score — grading first breeds leniency.
- **Never over-fit thin samples.** A calibration claim drawn from small n (under ~20) is itself marked uncertain; one miss is an anecdote, not a pattern.
- **An unscoreable prediction stays open as named debt** — never silently closed, never guessed at, never quietly dropped because a data source was down.
- **A belief read long after its last confirmation is re-verified before it is leaned on.** Confidence decays; the record doesn't say so by itself.

## Keeping the record honest about work

- **A commitment without a verifiable done-when is not tracked, it is hoped.** Track verifiable items; a fuzzy item creates the feeling of control without the substance.
- **One accountable owner per filed commitment or decision.** Diffuse ownership is the most common silent cause of a dropped thread — name the owner at filing time.
- **A baseline is data.** Plans-of-record and estimates are superseded, never edited, so "what did we believe in March" stays answerable and variance stays auditable.
- **Watch for watermelons.** A green-outside/red-inside status is a trust failure before it is a schedule failure — reward the early flag. And a generated roll-up can *launder* a watermelon: spot-check synthesised status against the primary artefact before filing it as truth.
- **Reversibility sets the ceremony.** A cheap, reversible merge call: decide fast. A destruction under retention, a structural migration: slow down, confirm, get it on the record.
- **An accepted gap beats a forced fix.** A low-stakes gap surfaced mid-crunch can be accepted — logged with an owner and a revisit trigger. A steward that only ever says "fix it now" is a justification engine, not a judgment engine.

## Source-map

Pointers, never frozen contents — re-check at the named trigger.

- **NISO Z39.19 / ISO 25964** (`niso.org`) — controlled-vocabulary construction; the thesaurus and cross-vocabulary-mapping standard. *Re-check when building or auditing a vocabulary or crosswalk.*
- **W3C** (`w3.org`) — SKOS, RDF/OWL: vocabularies and ontologies as linked data. *Re-check when publishing a vocabulary.*
- **DCMI / schema.org** (`dublincore.org`) — descriptive-metadata baseline; machine-readable structured metadata (JSON-LD). *Re-check when defining a metadata schema or tagging for machines.*
- **GO FAIR / FORCE11** (`go-fair.org`) — FAIR principles and maturity guidance. *Re-check when the corpus must serve machine retrieval or interchange.*
- **Nielsen Norman Group** (`nngroup.com`) — current empirical practice on card sorting, tree testing, findability. *Re-check before running a user-research method.*
- **Current RAG / GraphRAG / knowledge-graph literature** (vendor-neutral) — the fast-moving retrieval frontier. *Hold the pattern; pull the tool and benchmark at use time.*
- **Evaluation methodology** (LLM-eval practice, reason-then-grade rubrics, retrieval metrics) — fast-moving. *Pull the current technique when specifying a grader; never freeze a metric here.*
