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
- 2026-07-03: GitHub Pages deploy source switched from legacy branch-deploy to Actions-only (`build_type: workflow`), fixing a redundant dual-workflow setup where GitHub's own native Pages workflow was auto-deploying independently of and alongside the repo's custom "Check & Deploy" Action on every push — both could independently hit GitHub's transient deploy-infra errors, producing duplicate failure emails for a single hiccup. Confirmed FK cascade rules for `programs`/`program_phases`/`program_phase_workouts`/`workout_templates`/`client_programs`/`client_program_workouts` while researching a separate fix: `programs→program_phases→program_phase_workouts` and `workout_templates→workout_template_exercises` cascade automatically on delete; `client_programs→client_program_workouts` cascades too (0 orphaned rows found live); `workout_templates` itself does NOT cascade-delete when its `program_phase_workouts` row is removed (SET NULL) — the one real gap, now handled explicitly in `deleteProgram()`.
- 2026-07-03: coachapp GitHub auth hardened. A live `ghp_` PAT was found (and accidentally surfaced in-chat) embedded in plaintext in coachapp's `.git/config` remote URL; Jake deleted it, the embedded token was stripped from the remote (`git remote set-url` to the clean `https://github.com/jakendwest-ops/coachapp.git`), the stale cached credential was cleared, and git now authenticates to github.com via the `gh` CLI's keyring `gho_` OAuth token (`gh auth setup-git`; `repo` scope). No secret remains in `.git/config`; verified with an empty push (ec30ebf). The `vault.git` remote was already clean (no embedded token). Lesson banked: never run a command that echoes a known-embedded secret (`git remote -v`/`get-url`, `cat .git/config`).
