---
id: 2026-08-29-a-dead-preview-server-is-reported-as-failing-tests
status: open
priority: medium
reported: 2026-08-29
status_detail: "Found by hitting it: a full-suite run produced 39 identical ERR_CONNECTION_REFUSED failures that read as a catastrophic regression. Playwright has no webServer block and checks.sh has no server precondition, so a dead :3001 is reported as 'Playwright smoke tests failed -- Fix tests before pushing.'"
---

# A dead preview server is reported as failing tests, not as a missing server

**Found 2026-08-29 by hitting it during a `/save`.** The session-start ritual boots the :3001 preview
server; I went straight to running the suite without it (the previous session's server had died with
that session). Result: **39 identical failures**, every one
`page.goto: net::ERR_CONNECTION_REFUSED at http://localhost:3001/`.

My first read was that something was badly broken in the app. It took a process listing and a failure
tally to establish the real cause. **The output gives no signal that the dependency is missing rather
than the code being wrong** — and 39 red tests is exactly what a serious regression looks like.

## Two places, one gap

1. **`playwright.config.js` has no `webServer` block.** Playwright therefore neither starts the server
   nor verifies it, and `npm test` is not self-sufficient — it silently depends on out-of-band state.
2. **`scripts/checks.sh:337` (the pre-push smoke gate) has no server precondition either.** Verified: a
   grep of `checks.sh` for `3001`/`HttpListener`/`curl`/`server` returns **nothing**. With :3001 down,
   the gate fails all 57 smoke tests and prints:

   > `fail "Playwright smoke tests failed -- push blocked. Fix tests before pushing."`

   That message names the wrong cause. The tests are fine; the server is absent. Someone acting on it
   goes looking for a regression that does not exist.

**It fails safe, not open** — connection-refused cannot produce a false green, so nothing unsafe ships
because of this. The cost is misdirection and wasted runs, which is what it cost here.

## Why this is not "a problem that does not exist yet"

It happened, today, and produced a wrong initial diagnosis plus a discarded full-suite run. Per
[[feedback_no_speculative_fixes]] the bar is evidence, and this row is the evidence.

## Suggested fix (not yet done — needs its own pass)

A `webServer` block in `playwright.config.js` pointing at the same `HttpListener` command already
recorded in `.claude/launch.json`, with `reuseExistingServer: true` so it does not collide with a
server the session already booted (a second listener on 3001 just errors out — the `run-coachapp`
skill says so explicitly). That makes both `npm test` and the pre-push gate self-sufficient and moves
the failure message to "could not start the web server", which is the true cause.

**Assert the title, not just a 200.** `run-coachapp` already warns that a dead config can serve a
*different* app on 3001 — serving something and serving CoachApp are different checks. Any precondition
added here must assert `<title>CoachApp</title>`, or it becomes another check that passes without
looking. See [[feedback_reports_success_doing_nothing]].

**Closes when:** :3001 is deliberately stopped, and both `npm test` and `scripts/checks.sh` either start
the server themselves or fail with a message naming the missing server rather than the tests.
