---
id: 2026-08-14-test-gate-flakiness-returned-across-two-unrelated-files
status: open
priority: medium
reported: 2026-08-14
---

# The push gate started flaking again — 4 events in one day, 4 unrelated files

> **CORRECTED after a third full-suite run.** My first write-up of this pinned it on fixture isolation,
> because the first two events matched that shape. The third run made the real pattern visible: the
> DOMINANT signature is a **login timeout**, not shared data. Four events, four different files, and the
> only thing they have in common is that they happen deep into a 24-minute run. Fixture ordering is at
> most a secondary contributor. Corrected here rather than left half-right, because the wrong diagnosis
> would have sent the next session to rewrite fixtures and fix nothing.

**~~This is the fixture-isolation problem returning, in the two files that were deliberately NOT
converted on 2026-08-11.~~ — WRONG, see the correction above and the dose-response evidence at the
bottom. Left struck through rather than deleted, because the mistake is the lesson: it fit two data
points and it was the known problem, which is exactly why it was believed.** The 2026-08-11 decision
was nonetheless correct at the time and is recorded as such in
`2026-08-11-runner-spec-is-flaky-6-of-38-...md`: the gate had just been measured at **3 consecutive
`checks.sh` runs, 56 passed / 0 flaky**, so per Jake's standing rule of 2026-08-12 — *"do not change code
to fix a problem that does not exist yet"* — `programs.spec.js` and `client-workout.spec.js` were left
borrowing shared fixtures because nothing pointed at them.

**Today it exists.** Three separate events while shipping the 2026-08-14 screenshot-feedback work:

| Run | Result | Signature |
|---|---|---|
| Full suite #1 | 422 passed / **1 flaky** — `programs.spec.js:937` picker | render timing |
| `checks.sh` | 54 passed / **2 flaky** — `solo-account.spec.js:48` + `:116` | **login timeout** |
| Full suite #2 | 424 passed / **1 failed** — `solo-account.spec.js:48`, both attempts | wrong page (ordering) |
| Full suite #3 | 424 passed / **1 failed** `ledger-fixes-2026-07-30.spec.js:209` + **1 flaky** `:226` | **login timeout** |

**Four files, four runs, a different one each time — and `solo-account.spec.js:48` PASSED in runs #1 and
#3 while failing twice in #2.** No test is reliably broken; the gate is.

The dominant failure mode, in 2 of the 4 events (and the one that also blocked `checks.sh`):

    TimeoutError: page.waitForSelector: Timeout 15000ms exceeded.
      waiting for locator('#app-shell') to be visible
        33 × locator resolved to hidden <div id="app-shell" class="app-shell">…</div>
      at helpers.js:20  ->  loginAs / loginAsPT

The shell exists but never becomes visible, i.e. **`loadUserInfo` never completed** — sign-in itself did
not resolve. That points at **Supabase Auth rate-limiting or slowness**, which this project has hit before
(recorded in the 2026-07-25 session notes), not at test data. Every spec file calls `loginAsPT`/
`loginAsClient` in a `beforeEach`, so a 428-test run performs hundreds of sign-ins against one project;
the failures cluster deep into the run, which fits a rate limit far better than it fits fixture collisions.

The one event that was NOT a login timeout was full-suite #2, which got into the app and read the wrong
page — genuine ordering:

    Locator: locator('h1')
    Expected substring: "Library"
    Received string:    "My Training"

## It is ordering, not a regression — proven, not assumed

`solo-account.spec.js:48` was run four ways after the failure:

- **isolated** (`-g "solo can reach the Library"`) → passed
- **its own whole file** → 15 passed, 0 failed
- **paired with the new `screenshot-feedback-2026-08-14.spec.js`**, which sorts immediately before it
  alphabetically and was the obvious suspect → 24 passed, 0 failed
- **full suite** → failed twice

Same code, four different outcomes depending only on what ran before it. That is timing/ordering, and it
rules out the day's product changes: nothing in that diff renders the Library page or the solo nav.

## Why it matters more than the raw numbers suggest

A gate that fails at random eventually gets ignored — that is the whole reason the 2026-08-11 conversion
happened. Worse, **`checks.sh` is the pre-push hook**, so a flake here blocks a good push, and the reflex
after two false blocks is to stop reading the output. Today's `checks.sh` run reported "All checks passed"
*with 2 flaky underneath it*, which is exactly the shape that trains you not to look.

Note also that the two files implicated today are **not** the two named on 2026-08-11
(`programs.spec.js` was, `client-workout.spec.js` was not; `solo-account.spec.js` is new to the list). So
the conversion list was drawn from the evidence available then and is now known to be incomplete —
converting only the originally-named pair would not have caught `solo-account.spec.js`.

## Fix — in this order, because the diagnosis changed

**1. Auth first (the dominant signature).** Every spec `beforeEach` performs a fresh `signInWithPassword`,
so one full suite is hundreds of sign-ins against one Supabase project. Options, cheapest first:
- **Reuse a stored auth state** across tests (Playwright `storageState`) so the suite signs in a handful
  of times instead of hundreds. This is the standard fix and would remove the failure class outright.
- Confirm the theory first, and cheaply: instrument `loginAs` to capture the actual `signInWithPassword`
  error instead of only timing out on `#app-shell`. Right now a rate-limit and a genuinely broken login
  are indistinguishable in the output — which is exactly why this went 4 events without being diagnosed.

**2. Fixture isolation second** (real, but secondary — it explains 1 of 4 events). Same shape as the
`ownWorkout` fixture in `tests/fixtures.js` (2026-08-11): each test creates and tears down the rows it
needs instead of picking "whatever is first". Do NOT add a second fixture module — that is how these
drift. `solo-account.spec.js` additionally shares one login/view-switch state across 16 tests, so it may
need a `beforeEach` that re-asserts the Personal view rather than trusting the previous test.

**Do not start with (2).** That was my first instinct and the first version of this file said so; the
third run showed it would have rewritten a lot of fixtures and left the main failure class untouched.

**Deliberately not done on 2026-08-14.** It is a session of work on its own and the day's scope was Jake's
five reported items. Logged with the evidence so the next session starts from measurement, not memory.

**Closes when:** `checks.sh` runs three consecutive times at 0 flaky AND two consecutive full-suite runs
show 0 failed / 0 flaky. One green run is not evidence when the defect is non-determinism.

---

**STRONGER EVIDENCE, same day (session 2).** A full-suite run late in the session returned **1 failed +
11 flaky**, against 1-2 flaky earlier the same day. **All 12 failure signatures were identical:**

    TimeoutError: page.waitForSelector   (waiting for #app-shell, inside loginAs)

Not one product assertion failed. And the flake COUNT rose monotonically with the number of full suites
run in the session (~6 by that point, each performing ~430 sign-ins).

That dose-response is the diagnosis: a shared-fixture problem does not get worse the more times you run
the suite in a day, but a **rate limit** does. It also means late-session suite runs have degraded
signal — a run finishing green late in a heavy session is weaker evidence than the same run early on,
and failures must be CLASSIFIED (login-timeout vs product assertion), never just counted.

Practical consequence until this is fixed: prefer `checks.sh` (57 tests) as the trustworthy gate late in
a session, and treat a full-suite flake count as a measure of the session, not of the code.

## MEASURED 2026-08-26 — THREE files, and one proven non-deterministic

Four full-suite runs in one evening, same machine:

| run | code | progress-trend B4 | other |
|---|---|---|---|
| 1 | weight fix | **FAILED** | — |
| 2 | **identical to run 1** | **PASSED** | — |
| 3 | + Phase 1 | **FAILED** | ledger-fixes-2026-08-02 flaky (passed on retry) |
| 4 | + review fixes | PASSED | solo-account:142 flaky (passed on retry) |

**Runs 1 and 2 were byte-identical code with different outcomes** — non-determinism, not a regression.
The row title says “two unrelated files”; it is now **three**: `progress-trend.spec.js:5` (B4),
`ledger-fixes-2026-08-02.spec.js:6`, `solo-account.spec.js:142`.

Each was checked against the session’s own new fixtures before being called flaky — e.g.
`solo-account:142` asserts on the literal names `[E2E] PT-Only Lift` / `[E2E] Personal-Only Lift`,
which the new timestamped tags cannot collide with. All three use short visibility timeouts.

A plausible contributor is filed separately:
[[2026-08-26-e2e-specs-leave-client-rows-in-the-live-database]] — stranded fixture rows accumulating
on the real coach account.
