---
id: 2026-07-24-public-self-signup-closed-off-entirely
status: fixed-awaiting-jake
priority: critical
reported: 2026-07-24
status_detail: "fixed — awaiting Jake to flip the Supabase Auth dashboard setting"
---

# Public self-signup closed off entirely

✅ **FIXED + LIVE 2026-07-24 (57a188a) — Public self-signup closed off entirely.** Jake, live: *"I gave my brother the URL and from the login screen he was able to create an account. It looks like the system has automatically given him a PT account."* Confirmed as designed-but-unwanted: anyone with the URL got a full PT/coach account, no invite/approval gate. Removed the signup form, its handlers, and the show-signup/show-login toggle entirely (not hidden — `db.auth.signUp` is callable directly via devtools regardless of UI). **Jake still needs to turn off "Allow new users to sign up" in the Supabase Auth dashboard** separately — this closes the client-side path only. `tests/signup-removed-2026-07-24.spec.js` (3 tests).
