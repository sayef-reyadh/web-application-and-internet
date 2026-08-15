# Code Assessment Report

**Student:** Al Jubair Naiem
**ID:** 2022405
**Section:** Section 6
**Project:** Smart Blood Donation Emergency Platform
**Project Type:** Individual
**GitHub:** https://github.com/Al1622/Smart-Blood-Donation-Emergency-Platform

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/` — migrated from Flask to FastAPI, commit `444ac45` 07-31)
- Frontend: ✅ React confirmed (Vite, `frontend/src/`)

## Assessed at Commit
- **SHA:** `8fcb229` — 2026-08-11
- **Repo:** 34 commits, **10 active days** (2026-06-21 → 2026-08-11), issue/PR-based workflow

## Features Found
- Auth: signup (blocks self-assigned admin role), login, opaque bearer tokens, email verification
- Profiles: donor profile management with verification workflow (admin approves/rejects)
- Donor directory with blood-group/urgency filtering
- Emergency requests: create/list/close, urgency levels, blood-group validation
- Admin: profile verification, user management, **audit log**
- Frontend: AuthPage, AdminPage, DonorDirectoryPage, EmergencyPage, ProfilePage + Navbar, Vercel-deployed

## Security & Authentication (verified)
- ✅ **Argon2** password hashing (`argon2-cffi`, tuned time/memory/parallelism params)
- ✅ Opaque bearer tokens (`secrets.token_urlsafe`) stored server-side; `HTTPBearer` + `get_current_user` (DB lookup + `is_active` check → 403)
- ✅ **`require_admin`** dependency → 403 on admin routes; audit logging on admin actions
- ✅ CORS restricted to localhost + Vercel origins (`allow_origin_regex`)
- ⚠️ `role == "admin"` rejected at signup (guarded)
- ⚠️ **`.env.vercel` committed to the repo containing a live Vercel OIDC token** — credential exposure that should be revoked

## Data Persistence
- ✅ PostgreSQL (Neon, env `DATABASE_URL`, `postgres://` → `postgresql://` normalization) with SQLite fallback; SQLAlchemy models (User, Profile, EmergencyRequest, AuditLog)

## Runnability (tested in this session)
- ✅ **8/8 backend tests pass** (`test_verification_flow.py` — signup/login/verification flow)

## Observations
- Strong solo effort: real security stack, verification workflow, audit trail, live Vercel deploy, staged feature/PR history (issues #31–#36).
- Minor: docs nested `docs/docs/docs/...` paths; committed `.env.vercel` token.

## Future Scope
- Revoke and remove the committed Vercel OIDC token; flatten the docs directory; add a `.env.example`.

## Additional Code-Review Findings

- **One session per account, and tokens never expire.** `User.token` is a single column in [auth.py](repo/backend/app/auth.py); every login overwrites it (`user.token = create_token()` in [routers/auth.py](repo/backend/app/routers/auth.py)), silently invalidating the user's other devices, and the token carries no expiry — it remains valid until the next login or logout, with no rotation.
- **Legacy unsalted SHA-256 accepted at login.** `login()` falls back to `hashlib.sha256(password).hexdigest()` comparison before upgrading the hash — any account still on the legacy digest is protected only by a fast, unsalted hash that is trivial to brute-force if the database leaks.
- **Raw `dict` payloads instead of schemas.** `signup`, `login`, and `create_emergency_request` ([emergency.py](repo/backend/app/routers/emergency.py)) accept `payload: dict` rather than Pydantic models — no email-format validation, no password strength or minimum length (a one-character password is accepted), and no length limits on emergency `location`/`contact`/`description` fields.
- **Email verification is not enforced.** `signup` returns a working bearer token immediately, and `get_current_user` never checks `email_verified` — unverified accounts have full access, so the verification workflow gates nothing.
- **The audit sanitizer is a no-op.** In `log_audit()` ([auth.py](repo/backend/app/auth.py)), both branches of the `isinstance(value, (dict, list))` check do the identical `safe_metadata[key] = value` — only exact top-level key names are redacted, and any nested secret would pass straight through into the audit log.
- **DELETE doesn't delete.** `DELETE /api/emergency/{request_id}` performs a soft close (`is_active = False`) — a state transition masquerading as resource deletion, which breaks REST semantics for any client.

## Detailed Feedback (Instructor Review)

**What you did well.** This is one of the strongest submissions in the cohort. Argon2 hashing with tuned parameters, opaque bearer tokens backed by server-side DB lookups with an `is_active` check, `require_admin` enforced on all six admin routes, audit logging on admin actions, a donor verification workflow, signup that blocks self-assigned admin, restricted CORS, 8/8 passing backend tests, a live Vercel deploy, and a disciplined issue/PR history including a mid-project Flask-to-FastAPI migration. The opaque-token design is not a signed JWT, but it is implemented correctly and is secure.

**Where to grow.** You committed `.env.vercel` containing a live Vercel OIDC token to a public repository. That is a real credential exposure — exactly the kind of mistake your otherwise-excellent security work is supposed to prevent — and deleting the file is not enough because the token lives in git history. Your docs are also buried under nested `docs/docs/docs/...` paths, which is sloppy packaging for an otherwise professional repo.

**future scope ideas:** revoke the exposed Vercel token immediately, purge it from history, add a `.env.example`, flatten the docs directory, and consider adding token expiry/rotation to your bearer tokens.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
