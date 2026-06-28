# Research & Intelligence — craft

_For future Vision: load this before any research task — multi-source investigation, market or competitor intelligence, keyword/SERP work, claim verification, or any briefing the owner will act on._

Senior layer. It assumes the basics (cite primary sources, triangulate, lead BLUF, never trust one link) and adds the structure and judgment on top: which frame to reach for, how to calibrate a call, where the work fails, where to pull the live fact. Volatile specifics — tool rankings, benchmark scores, detector accuracy — are deliberately not frozen; see the source-map at the bottom and re-check at the named trigger.

---

## Mental models

- **The Intelligence Cycle** — Direction → Collection → Processing → Analysis → Dissemination, feedback re-entering at the top (it iterates, never a straight line). Use it to locate where a job actually sits: most "research failures" are mis-scoped *Direction*, not weak collection.
- **Question → decision, worked backwards.** Start from the decision the answer serves → the judgement that decision needs → the specific answerable sub-questions → only then collection. This is what stops a "scan" becoming a fishing trip; the deliverable is a *decision input*, not a topic dump.
- **Analysis of Competing Hypotheses (ACH).** Enumerate the full plausible set up front (mutually exclusive, testable, including the uncomfortable one), score each piece of evidence by **diagnosticity** (does it discriminate *between* hypotheses, not merely fit one?), then work to **disconfirm**, not confirm. Reach for it when stakes are high, deception is possible, or you feel early conviction.
- **Key Assumptions Check.** Surface the load-bearing assumptions *before* they silently drive the conclusion. Cheapest high-yield technique — run it first, every time.
- **Base rate before the case.** Set the estimate against the reference-class frequency *first*, then update on case specifics; don't reason from vivid particulars and back into a number. Base-rate neglect is the quiet engine under most over-confidence.
- **Two-tier grading: source ≠ information.** Grade the *source's reliability* and the *claim's credibility* separately and in isolation (the Admiralty A–F / 1–6 matrix), so a trusted source's unconfirmed claim isn't laundered into fact, and a weak source's confirmable claim isn't dismissed.
- **Confidence is two axes, not one.** *Likelihood* the thing is true vs *analytic confidence* in your evidentiary base are independent — "likely but low-confidence" is a coherent, honest call. Communicate both with consistent estimative-probability bands; never a false-precision point number, never collapsed into one phrase.
- **Fact / inference / assessment are different objects.** What a source *says* ≠ what you *conclude* ≠ your confidence-weighted *judgement*. Conflating them is the most common way a brief misleads while every line is "true."

## Playbooks

- **Scoping a tasking:** decision served → key judgement needed → answerable sub-questions → a collection plan against each → stop-criterion defined *now* (what answer ends the search). End at a tasking, not a vibe.
- **Collection → verification, in that order:** cast wide to discover candidates, then *collect → check → verify*, never collapsed into one step. Convergence is reached when the sub-questions are answered, not when you run out of tabs.
- **SIFT → CRAAP, layered.** SIFT (Stop · Investigate the source · Find better coverage · Trace to original) as the fast pre-filter on anything web- or AI-surfaced; escalate to CRAAP (Currency · Relevance · Authority · Accuracy · Purpose) only when a source survives SIFT and the stakes warrant depth. Don't deep-evaluate a source you should have discarded at "Investigate."
- **Lateral reading for contested or AI-surfaced claims.** Don't read one source deeply — open independent tabs and corroborate from *unrelated* origins. Discovery and verification are different motions: a tool (or AI) discovers; lateral reading verifies.
- **Trace the citation chain to the root.** For any load-bearing claim, follow it back: three sources echoing one study are **one** source. True independence = separate data, methodology, or primary access — not separate URLs.
- **Filter before hydrate.** On a wide retrieval, rank and discard on cheap signals (title, source, snippet, recency) *before* spending the cost to fetch and read full sources. Iterative evaluate → refine: re-run the query against what the first pass revealed rather than reading everything once.
- **Synthesis:** lead BLUF → state the judgement with calibrated confidence → show the evidentiary basis → flag explicitly what is thin, contested, or assumed — and name the **open questions** as part of the deliverable, never as silent gaps. The product answers the decision; it is not a chronological dump of what you found.

## The bounded research loop

For an open-web investigation that has to converge rather than sprawl, run it as a capped loop. State the expected cost first when the run is large (rounds × angles × fetches) and confirm before spending it.

- **Round 0 — entity resolution.** Before any search, resolve every named entity in the topic (a company, person, market, lender) to its **canonical identity**: official/registered name, ticker or company number, known aliases and handles. One cheap pass up front stops every subsequent angle querying a near-namesake. Skip only when the topic names no specific entity. Round 0 doesn't count against the cap.
- **Round 1 — broad.** Decompose the topic into 3–5 *non-overlapping* angles; per angle, a few searches and the top 2–3 fetches that survive the filter. Extract claims, entities, and open questions from each.
- **Round 2 — named-gap fill.** Score what came back and **name what's still missing** — contradictions, holes, thin spots. The next round targets those named gaps specifically, never a re-broadening. A round that can't name its gaps is finished.
- **Round 3 — conditional.** One final targeted pass only if material contradictions or holes remain; otherwise go straight to synthesis. **Hard cap: 3 rounds.** Quality over volume — 30 pages of noise is a failure, not a success.
- **Carry the purpose, not just the query.** Every search and fetch carries *why* it runs — the gap it closes — so a result is judged on whether it advances the purpose, not on keyword match alone.
- **Cluster the same story.** The same story found across Reddit, X, and an article is **one** clustered finding with all sources cited — never three near-identical entries. Where a source carries an engagement proxy (upvotes, replies, views), weight by it alongside recency; social traction is signal in the content lane.
- **A failed fetch is logged and skipped, never fatal.** The loop continues and the missing source surfaces in the synthesis's open questions. Never present a silent gap as completeness.

## Where senior diverges from junior

- **Diagnosticity beats volume.** Ten sources that fit every hypothesis move you nowhere; one that *eliminates* a hypothesis is worth more than the ten. Weigh evidence by what it rules out.
- **Strong claim + weak sourcing → escalate verification, not confidence.** The instinct to round a striking, thinly-sourced finding *up* is exactly backwards; surprise is a trigger to dig, not to believe.
- **If you can't name what would disprove it, you haven't tested it.** A hypothesis with no conceivable disconfirming evidence is a belief, not a finding. Specify the falsifier before you commit.
- **Calibrate, don't hedge.** "Possibly" used to mean "I didn't check" is a tell of junior work. Assign an *honest* band and say *why it's that band* — including, when warranted, "high confidence" or a clean "we don't know."
- **Hold the line, then update on evidence — neither reflex.** Revise a standing judgement when genuinely new diagnostic evidence arrives, not on the latest headline and not never; name *in advance* what would move it. Knee-jerk flipping and anchored refusal-to-update are the same failure of discipline.
- **Absence of evidence ≠ evidence of absence** — *unless* you'd expect to see it and reliably don't (then absence is itself diagnostic). Mark a genuine unknown as unknown; never silently fill it.
- **Match rigour to stakes and reversibility.** A reversible, low-stakes input gets fast triangulation; an irreversible or high-consequence judgement earns full ACH + active disconfirmation + a second independent chain. Over-verifying a throwaway and under-verifying a bet-the-quarter call are the same error.
- **Independence is the scarce resource, not quantity.** The question is never "how many sources" but "how many *independent* ones" — the same wire story on 40 sites is one source.
- **Provenance over plausibility for media.** A clean-looking image/video/audio proves nothing about origin; treat synthesis as the default hypothesis for any consequential media and pull provenance signals before relying on it.

## Failure catalogue

- **Confirmation bias** — collecting only what fits the prior. The master failure; ACH, devil's advocacy, and premortem exist to counter it.
- **Citation laundering / false independence** — echoes of one origin counted as corroboration; the single most common way a well-cited brief is still wrong.
- **Boiling the ocean** — no defined sub-questions, so collection never converges and the deadline arbitrates instead of the question.
- **Volume-as-confidence** — mistaking a thick evidence pile for a strong one; anchoring on the first or best-*written* source.
- **Base-rate neglect** — reasoning from the vivid case, ignoring the reference-class frequency.
- **Mirror-imaging** — assuming the subject reasons, values, and constrains as you would. Lethal in adversary and competitor work.
- **Anchored non-updating** — clinging to a standing line after disconfirming evidence has arrived; the mirror of premature convergence, just as costly.
- **Source-reliability bleeding into claim-credibility** — "they're usually right, so this is true." Two-tier grading exists to stop exactly this.
- **Recency bias** — newest ≠ truest; a fresh blog can be wrong where an old primary is right.
- **Treating a scout as an arbiter** — taking an AI/aggregator/secondary's synthesis as the finding instead of as a lead to verify.
- **Premature convergence / satisficing** — stopping at the first coherent story that fits, before testing a competing one.
- **Wrong-entity research** — angles querying a near-namesake because nobody resolved the canonical identity first; cheap to prevent, expensive to discover late.
- **Burying the judgement** — a chronological dump with no BLUF, no confidence, no thin/contested flags: technically complete, operationally useless.

## The AI frontier

The fast-moving edge — frameworks and judgment, not this month's tool ranking.

- **AI as scout, never arbiter (the root rule).** LLM deep-research assistants and agentic search are discovery accelerants; every AI-surfaced claim is *unverified* until confirmed at the primary source. The durable skill migrates *away* from discovery (now cheap) and *toward* verification, framing, and calibration.
- **RAG faithfulness ≠ retrieval quality.** Even a well-curated pipeline produces faithfulness failures — outputs that contradict or over-reach their own retrieved sources, including **fabricated citations**. A cited AI answer is not a verified one: check that each claim is actually *supported by* the span it points to, not merely that a citation exists.
- **Provenance as a first-class signal.** For consequential media, default to "could be synthetic" and seek content provenance (C2PA / Content Credentials, model watermarks). Provenance is a *signal, not proof* — metadata is routinely stripped by screenshots and re-encodes, its *absence* proves nothing, and a detector score is probabilistic. Triangulate with reverse-image, geolocation, chronolocation.
- **Adversarial information environment.** Assume coordinated inauthentic behaviour, seeded narratives, and AI-generated source-flooding as a *baseline*, not an edge case — engagement and apparent consensus are cheaply manufactured. Rank by independent corroboration, not reach.
- **Audit-trail discipline as the moat.** As synthesis automates, the differentiator is a *traceable* chain from judgement → claim → primary source that survives scrutiny. The trail is the trust.

## Worked reasoning traces

- **One striking, on-thesis stat appears early.** Surprise + perfect-fit + early-conviction is the confirmation-bias trap. Trace the chain — three "independent" cites resolve to one un-peer-reviewed blog. → Treat as **one** weak source; find a truly independent primary or down-weight to low confidence and flag it thin. *Volume looked like corroboration; diagnosticity and origin-tracing dissolved it.*
- **An AI deep-research tool returns a fully-cited, fluent briefing under deadline.** A citation is not support; RAG pipelines fabricate citations and over-reach. Spot-check load-bearing claims against the cited spans — two of five top claims aren't in the linked source. → Use the AI output as a *lead map*, verify each material claim at primary, ship only what survives. *The AI is the scout; the verification step is yours.*
- **A consequential photo/video carries a judgement.** Default hypothesis: synthetic or out-of-context until shown otherwise. Pull provenance *and* run reverse-image + geolocation. No provenance metadata present → absence is **not** evidence of fakery (strips routinely) and presence wouldn't be proof. → Rest the judgement on *convergence* of independent verification, hold confidence moderate, state the basis.
- **High-stakes "what's really going on" with a contested, deception-plausible subject.** The ACH case: enumerate the full set, score by what it *rules out*, seek disconfirmers, watch for mirror-imaging. One hypothesis survives best but evidence is thin → report it as the leading judgement at explicitly-moderate confidence, name the runner-up and what would flip it. *The honest product is a ranked, disconfirmation-tested set — not one confident story that ignored the alternatives.*
- **The owner wants a deeper dive on a low-stakes, reversible input.** First-pass triangulation already converged across independent sources; no deception incentive; a marginal hour of ACH wouldn't change the call. → **Stop**: declare the finding at the confidence already earned, note it's lightly-verified and reversible, don't spend the rigour. *Over-verifying the trivial is the same error as under-verifying the critical; research that only ever says "dig deeper" is a cost engine.*

## Authoritative source-map

WHERE to pull the current fact — pointers, not copies. Re-check at the named trigger; never freeze the contents.

- **Standard tradecraft texts** — Heuer & Pherson, *Structured Analytic Techniques*; Heuer, *Psychology of Intelligence Analysis* — the durable method (ACH, key-assumptions, bias countermeasures). *Re-check edition on a new release; the method is stable.*
- **Collection / cycle doctrine** — JP 2-0 and agency intelligence-process doctrine — the authority for the Intelligence Cycle itself. *Stable; re-check only on a doctrinal revision.*
- **ODNI Intelligence Community Directives** (`intelligence.gov` / `dni.gov`) — ICD 203 (analytic standards: objectivity, sourcing, calibrated confidence), 206, 208 — *how to express* a judgement and its confidence. *Re-check on a directive revision.*
- **Admiralty / NATO grading scheme** (any current tradecraft reference) — the A–F / 1–6 source-vs-information matrix. *Stable; re-check only descriptor wording if it matters.*
- **CRAAP / SIFT originators & current digital-literacy guidance** (Caulfield's SIFT; university research-guide pages) — live phrasing and AI-era adaptations of source evaluation. *Re-check for AI-content-specific updates.*
- **Current AI deep-research / RAG-evaluation literature** (arXiv cs.CL/cs.IR; RAG faithfulness & hallucination benchmarks; vendor leaderboards) — which tool leads, current accuracy/faithfulness scores, span-verification methods. *Highly volatile — re-check before citing any tool ranking, benchmark, or "tool X now does Y."*
- **Content-provenance standards & detector landscape** (`c2pa.org`; Content Authenticity Initiative; model-watermark docs) — provenance spec editions, adoption coverage, detector accuracy. *Volatile — re-check before relying on a specific tool, coverage claim, or version.*
- **OSINT technique directories** (the OSINT Framework taxonomy; reputable practitioner orgs) — the *categories* of collection technique and current tool names. *Re-check tool names on use; the taxonomy is durable, the tools churn.*
- **The applicable legal/ethical boundary for collection** (jurisdiction-specific privacy/data-protection law; platform ToS) — what collection is *lawful and in-scope*, not just technically possible. *Re-check at task start for the jurisdiction and platforms in play — a legal duty isn't the researcher's to infer.*
