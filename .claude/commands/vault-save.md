---
description: Consolidate the session into the vault (renamed from /save — the bare /save verb belongs to the CoachApp end-of-session skill)
---

# /vault-save — the deposit ritual

> Renamed from `/save` on 2026-07-02. The bare `/save` verb is owned by the account-level CoachApp end-of-session skill (`~/.claude/skills/save/`) — a different, legitimate system. This ritual is the Vault-OS deposit; invoke it as `/vault-save`.

Move the session's value from chat to disk. Run the full ritual at session end; a light mid-session checkpoint is steps 2, 3 and 6 — escalate to the full ritual the moment a decision or new analysis warrants it. If no project is loaded, ask which one this saves to before touching anything.

1. **Sweep for unverified claims.** Anything this session asserted as done or true without an actual check: verify it now, or bank it carrying `UNVERIFIED` with the reason. Nothing false gets deposited.

2. **Rewrite STATUS.md wholesale** — live state plus the continuity block, per `System/conventions.md`. Anything no longer live moves to LOG; STATUS is never a history.

3. **Append a dated LOG entry** — what got done, decided, changed. Decisions carry Why and Revisit-if (formats: conventions).

4. **Prediction scan — mandatory.** Re-read the session for every checkable judgment made: a recommendation, a forecast, a call reality will later grade. Each becomes one line in `Vault/memory/predictions.jsonl` with a `verify_by` date (record shape: conventions). This is the learning loop's intake — skipping it is the one lapse the OS cannot afford. "No predictions this session" is a legitimate finding; not looking is not.

5. **Owner-voice & deltas — sweep, don't skip.** Two signals, looked for every save (a blessing-gated duty; no deep profile, no sweep):
   - **Chat-voice deltas.** Scan the owner's *own* messages this session for voice signal — openings, sentence shape, punctuation/casing habits, vocabulary, how he flags a half-formed idea, how he gives instructions. Bank only what's *new* into `Vault/owner/voice.md`, **register-tagged**: chat-to-Vision evidence updates the *casual/AI* slice and Observed Signals only — never the public/formal slice (his chat register is not his output register; letting it leak corrupts how you draft his posts and emails). Distil the pattern, don't paste transcript. "No new voice deltas" is a written finding, not silence.
   - **Draft→ship diff.** If you drafted anything this session the owner then edited before it went out, the diff is the highest-value voice signal there is — bank what he cut, added, reworded, and the direction of it, into the matching register of `voice.md`. If it reveals a reaction pattern, that's also a `kind: owner` prediction at step 4.

   Then, **when the session earned it** — judgment, not checklist:
   - a durable claim about the owner's world → `Vault/memory/beliefs.jsonl`
   - a lesson that will still be true next month → `Vault/memory/lessons.jsonl`, through the do-not-capture filter
   - a behaviour or taste delta about the owner (a rejection, a preference shown, a reframe) → the relevant `Vault/owner/` file
   - a question answered well that future Vision will face again → distil into the project's `analysis/`

6. **Append the session-end line** to `Vault/ledgers/log.md` via shell `Add-Content` — editor writes to ledgers are denied.

7. **Confirm in one line**, ending with the next action.
