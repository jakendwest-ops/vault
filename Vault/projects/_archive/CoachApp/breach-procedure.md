# CoachApp — Personal Data Breach Procedure (UK GDPR / ICO)

**Status:** Active from 2026-07-12. Owner: Jake West (sole controller).
**Not legal advice.** This is an operational runbook. For a breach that is clearly high-risk or
large-scale, take the containment steps below *and* seek professional/legal advice in parallel.

CoachApp processes **special-category health data** (bodyweight, body-fat, progress photos,
performance metrics) about coaches' clients. That raises the risk profile of any breach: the bar for
"likely to result in a risk to rights and freedoms" is lower than for ordinary contact data.

---

## 0. The one number to remember: 72 hours

If a breach is notifiable, the ICO must be told **within 72 hours of you becoming *aware*** of it —
not 72 hours from when it happened. "Aware" means you have a reasonable degree of certainty that a
security incident occurred that compromised personal data. The clock includes weekends. If you report
late, you must explain the delay.

You do **not** have to have the whole picture in 72 hours. You can make an initial report and follow
up with details ("phased notification"). Under-reporting on time beats a perfect report that is late.

---

## 1. What counts as a personal data breach

A breach of security leading to the **accidental or unlawful destruction, loss, alteration,
unauthorised disclosure of, or access to** personal data. Three flavours — all count:

- **Confidentiality** — data seen by / disclosed to someone who shouldn't see it.
  *CoachApp examples:* an RLS/storage policy lets one coach read another's client data; a bucket
  goes public; the wrong client's records render on a shared page.
- **Integrity** — data altered without authorisation.
  *CoachApp examples:* a cross-tenant UPDATE/DELETE policy lets one user change/delete another's rows.
- **Availability** — data lost or destroyed and not recoverable.
  *CoachApp examples:* a bad migration or a delete-guard bug destroys client workout history with no
  backup.

A **vulnerability that was open but not actually exploited** is not automatically a notifiable breach,
but it **must still be logged** (§5) and assessed (§2). The question for notification is whether
personal data was actually accessed/affected, and the risk that follows.

---

## 2. The decision: is it notifiable?

Two independent questions.

### 2a. Notify the ICO? — unless the breach is *unlikely* to result in a risk to individuals.

Notify **unless** you can justify that it is unlikely to result in a risk to people's rights and
freedoms. With health data involved, lean towards notifying. Consider:

- **Was real personal data actually accessed/affected**, or only a test/synthetic account?
- **Whose data, and how much** — one dev test photo vs. every client's photos.
- **Who could have accessed it** — a real third party, or only accounts you control?
- **Special category?** Health data → higher risk.
- **Consequences** — could it cause distress, embarrassment, discrimination, identity fraud?
- **Mitigation already in place** — was the data encrypted / already deleted / never actually reachable by a real party?

If unlikely-to-risk: **do not notify ICO, but record the incident and your reasoning** (§5).
If in genuine doubt: **notify.** The ICO does not penalise a good-faith over-report; it does penalise
a missed one.

### 2b. Notify the affected individuals? — only if **high risk** to them.

If the breach is likely to result in a **high** risk to individuals, tell them **without undue delay**,
in plain language: what happened, likely consequences, what you're doing, and how they can protect
themselves (e.g. what to watch for). You can avoid direct notification only if you've since rendered
the data unintelligible (e.g. strong encryption), or it would take disproportionate effort (then use a
public notice instead). For CoachApp's scale, direct email to affected clients is expected if it
reaches this bar.

---

## 3. First hour — contain, then assess (do these in order)

1. **Stop the bleed.** Revoke the leaking access *now*, before diagnosis is complete. For an RLS/
   storage leak that means fixing/dropping the offending policy (see `scripts/fix-storage-rls-*.sql`
   for the pattern) or, if faster, taking the affected surface offline. Containment is not delayed by
   investigation.
2. **Prove containment.** Re-run the behavioural probe that found it (rls-audit.spec.js /
   storage-privacy.spec.js) and confirm it goes green. A fix you haven't re-tested is a hope.
3. **Record the "aware" timestamp.** Note the moment you became aware — the 72h clock starts here.
4. **Scope it.** What data, whose, how many people, for how long was it exposed, and **is there any
   evidence it was actually accessed by a party who shouldn't** (vs. merely reachable)? Supabase
   dashboard → Logs → API/Storage is the evidence source.
5. **Assess** against §2 and decide: ICO? individuals? neither-but-logged?
6. **Log it** (§5) regardless of the decision.

---

## 4. Reporting to the ICO — how

- **Online:** ICO Personal Data Breach report form at `ico.org.uk` (search "report a breach"), or
  **phone 0303 123 1113** for help / to report.
- **Have ready:** what happened and when; how you became aware; categories and approximate number of
  **data subjects** affected; categories and approximate number of **records**; likely consequences;
  measures taken or proposed; your contact details as controller. Unknowns are fine in an initial
  report — say "estimated / to follow".

---

## 5. The internal breach log — MANDATORY, always

UK GDPR requires you to **document every personal data breach**, including ones you decide **not** to
report — the facts, effects, and remedial action. This log is what demonstrates compliance if the ICO
ever asks. Keep it in the Vault (append here, or a dedicated `breach-log.md`). One entry per incident:

```
### YYYY-MM-DD — <short title>
- Became aware:        <timestamp>
- What happened:       <the failure, in one paragraph>
- Data involved:       <categories; special category y/n>
- People affected:     <count / "none — test data only">
- Actually accessed?:  <evidence of real access, or "vulnerability only">
- Containment:         <what was done, timestamp> + <proof: test green>
- ICO notified?:       <yes+ref / no + one-line justification>
- Individuals told?:   <yes / no + justification>
- Root cause + fix:    <link to commit / SQL>
- Follow-up:           <new test / check added so it can't recur silently>
```

---

## 6. Worked example — the 2026-07-12 storage leak (why it was NOT notified)

Shows the judgement in action; the answer here is "log, don't notify", and the reasoning is what
matters.

- **Became aware:** 2026-07-12, during a `/deploy-check`, when `storage-privacy.spec.js` proved a
  second coach (owns nothing) could download and delete any client's progress photos.
- **What happened:** three `storage.objects` policies on `progress-photos` were scoped by `bucket_id`
  alone — any *authenticated* user could read/delete/write any photo. (Bucket was still `public =
  false`, so anonymous strangers were refused — which is why the old flag-only check missed it.)
- **Data involved:** progress photos = special-category health data. In principle, high risk.
- **People actually affected:** **none.** The only real object in the bucket was Jake's own dev-era
  test photo. No real second coach existed on the system — the only other coach account was the test
  coach created minutes earlier by the audit itself.
- **Actually accessed by a real party?** No. The only cross-tenant access was the test harness
  demonstrating the hole. No real client's data was reachable by any real third party, because there
  were no other real tenants.
- **Containment:** dropped the three policies (`scripts/fix-storage-rls-2026-07-12.sql`); re-ran
  `storage-privacy.spec.js` → green (cross-tenant download refused, planted victim). Deleted the one
  orphan dev photo.
- **ICO notified?** No — assessed as unlikely to result in a risk to any real individual, because no
  real data subject's data was accessed or accessible by a real party. Pre-beta, single-tenant-in-
  practice.
- **Individuals told?** No — no affected individuals exist.
- **Follow-up:** extended the behavioural audit to cover Storage (previously a blind spot in both the
  harness and `/deploy-check`); replaced deploy-check's `buckets.public` config-read with the probe.

**The point:** the same bug, discovered *after* real coaches are on the platform, would very likely be
**notifiable** (special-category data, cross-tenant, real people). What made it non-notifiable was
purely that no real data subject was affected. That distinction is exactly what §2 exists to draw —
and it is why this must be re-assessed, not assumed, every single time.
