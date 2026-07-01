---
bi_temporal: true
---

# CoachApp — CRITICAL facts

_Always-relevant facts. Changes append to the timeline block at the bottom — never overwrite._

---

## Infrastructure

| Fact | Value |
|---|---|
| Supabase project ID | `avilxuiacmtgeoxxhfhc` |
| Supabase region | eu-west-1 (Ireland) — adequate for UK GDPR |
| GitHub Pages URL | https://jakendwest-ops.github.io/coachapp |
| GitHub repo | jakendwest-ops/coachapp |
| Vault backup repo | jakendwest-ops/vault |

---

## Storage buckets

| Bucket | Visibility | Expiry | Notes |
|---|---|---|---|
| `logos` | Private | 604800s (7 days) | PT logo upload; MIME-restricted to image types; 5 RLS policies |
| `progress-photos` | Private (fixed 2026-06-28) | 3600s (1hr) | Client body photos; use `createSignedUrls` batch; 2 RLS policies |

**Rule: all buckets must be private. No exceptions.**

---

## GDPR compliance status (2026-06-28)

| Requirement | Status |
|---|---|
| Consent at signup | ✅ Checkbox, validated in handler |
| Data export | ✅ `downloadMyData()` in Settings |
| Right to erasure | ✅ `delete_current_user()` RPC + Settings UI |
| Data residency | ✅ eu-west-1 (Ireland) |
| Privacy policy page | ✅ `/privacy-policy.html` live (v158, 2026-06-29); consent checkbox links to it |
| ICO breach notification | ❌ Process not documented — needed before beta |

---

## Security constraints — non-negotiable

1. **No PUBLIC storage buckets** — use signed URLs always
2. **No PII in log calls** — IDs and dates only; pre-push hook enforces
3. **RLS on every table** — enabled before first INSERT; never `qual = 'true'` on sensitive tables
4. **No `auth.users` in RLS policies** — use `auth.uid()` / `auth.email()` only
5. **Health data = special category** — weight, body fat, photos, performance metrics
6. **New table checklist** — RLS + add to `downloadMyData()` + add to `delete_current_user()` RPC

---

## Key DB functions

| Function | Purpose |
|---|---|
| `delete_current_user()` | GDPR erasure — deletes all user data then auth.users row; security definer |

---

## Timeline

- 2026-06-28: GDPR hardening session. progress-photos made private. PII stripped from 16 log sites. Consent checkbox added to signup. Data export + delete account built. Storage bucket RLS established for logos and progress-photos.
- 2026-07-01: Found and fixed a solo-account RLS gap — `client_1rms` had no INSERT/UPDATE/DELETE policy for solo users, and `client_programs` had no UPDATE/DELETE policy for solo users (both only had coach-scoped write policies + a partial solo insert/select). Added 5 solo-scoped policies matching the `client_program_workouts` pattern. Lesson: when adding a new role/account type, every table it touches needs its own explicit write policies checked — a working INSERT does not imply UPDATE/DELETE also work.
