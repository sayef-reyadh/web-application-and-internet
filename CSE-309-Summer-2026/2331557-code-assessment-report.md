# Code Assessment Report

**Student:** Mosfiqa Akter Sohely
**ID:** 2331557
**Section:** Section 6
**Project:** FoodBridge — Food Rescue & Donation Platform
**Project Type:** Individual
**GitHub:** https://github.com/mosfiqa/Food-Rescue-Donation-Platform

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/`, 7 router modules)
- Frontend: ✅ React confirmed (Vite, `frontend/src/`, 9 pages)

## Assessed at Commit
- **SHA:** `570f61f` — 2026-08-10 — "Merge GitHub main with FoodBridge project"
- **Repo:** 19 commits, **9 active days** (2026-06-13 → 2026-08-10), staged via PRs (#29 frontend setup, #30 FastAPI backend, #34 Add-Donation frontend, #35 donation-creation backend)

## Features Found
- Auth: register/login/me/profile-update, roles donor/ngo/volunteer (Literal — **admin cannot self-register**)
- Donations: create/list/search (food type, status, mine-scoped), update, status workflow (available → requested → delivered/expired), auto-expiry refresh
- NGO requests on donations, volunteer assignments, notifications + notification settings, admin dashboard/user management/reports
- Frontend: 9 Vite pages (Auth, Dashboard, Donations, Requests, Assignments, Notifications, Profile, Users, Reports), role-based UI, `api.js` fetch wrapper with Bearer token + error normalization

## Security & Authentication (verified)
- ✅ Password hashing: **PBKDF2-SHA256, 210,000 iterations, random 16-byte salt**, `hmac.compare_digest` verify
- ✅ Real JWT (PyJWT, HS256, `iat` + `exp`), 480-min expiry via env
- ✅ `HTTPBearer` + `get_current_user` (token decode, **DB re-query**, `is_active` check)
- ✅ `require_roles(*roles)` dependency → **403** on insufficient role
- ✅ Role-scoped queries (donor sees own donations, ngo sees own requests, volunteer sees assignments)
- ✅ Registration role `Literal["donor","ngo","volunteer"]` — no privilege escalation
- ⚠️ `secret_key` default `"foodbridge-development-secret-change-me"` (env-overridable)

## Data Persistence
- ✅ SQLAlchemy 2.0 + SQLite (FK pragma ON), models with relationships + StatusHistory, seed script, Pydantic schemas

## Runnability (tested in this session)
- ✅ **8/8 backend tests pass** (auth flow, role guards, donation lifecycle, notification settings)

## Observations
- Compact but complete solo full-stack project: layered backend (routers/services/security/dependencies), 8 passing tests, pytest.ini, .env.example, QA docs (VALIDATION.md, FIXES.md, DASHBOARD-UPDATE.md), pinned requirements, PowerShell/shell dev scripts, per-phase PRs.

## Future Scope
- Require `FOODBRIDGE_SECRET_KEY` from env in production; consider alembic migrations (currently `create_all`); commit history is upload-based ("Add files via upload") — smaller, message-annotated commits would improve the repo trail.

## Additional Code-Review Findings

- `seed_database()` runs unconditionally in the FastAPI `lifespan` handler (`backend/app/main.py`) and creates accounts with hardcoded, publicly visible passwords (`Admin123!`, `Donor123!`, … in `backend/app/seed.py`). It is idempotent only in the sense that it skips when any user exists — a fresh production deployment would silently go live with a **known admin credential**. Gate seeding behind an explicit debug/env flag.
- Expiry is enforced lazily with a write inside a read: `_refresh_expired()` in `backend/app/routers/donations.py` mutates and commits donation statuses during `GET` requests. This means statuses in other views (dashboard stats, admin reports) can be stale until someone happens to list donations, read endpoints carry surprise write transactions, and concurrent listers can race the commit. A scheduled job or a status computed at read time would be cleaner.
- `GET /api/donations` has no pagination — it returns the entire filtered table (with `joinedload` of donors) on every call, and every call also runs the full expiry scan above. This will not scale past demo data.
- The access-token lifetime defaults to 480 minutes (8 hours, `backend/app/config.py`) and the token is stored in `localStorage` (`frontend/src/services/api.js`, key `foodbridge_token`) with no refresh or server-side revocation — a stolen token stays usable for a full workday. Consider shorter expiries plus refresh tokens, or httpOnly cookies.
- Repo hygiene: the git index contains junk artifacts at the root — a 2-byte file literally named `my files`, a duplicate `requirements (1).txt`, a stray `backend-package.json`, and a root `package.json` in a Python backend project — consistent with the upload-based commit workflow already noted. Clean these up and rely on `backend/requirements.txt` / `frontend/package.json` as the single sources of truth.
- Password policy is length-only (`min_length=8`, `max_length=128` in `backend/app/schemas.py`) — no complexity or common-password checks, so `password` would be accepted if it were 8+ chars.

## Detailed Feedback (Instructor Review)

**What you did well.** This is one of the strongest solo submissions I reviewed. Your security stack is real and verifiable: PBKDF2-SHA256 with 210,000 iterations and random per-user salts, signed PyJWT tokens with issued-at and expiry claims, a `get_current_user` dependency that re-queries the database, and a `require_roles` guard that returns 403. Role-scoped queries and a registration form that cannot mint admins show mature security thinking. The donation lifecycle, NGO requests, volunteer assignments, and notifications are all wired end-to-end, and all 8 backend tests pass.

**Where to grow.** Remove the hardcoded default secret and require it from the environment in production. Adopt alembic migrations instead of `create_all`. Your commit history leans on "Add files via upload" — use small, well-messaged commits so your work stays auditable.

**Attribution note.** Individual project; the staged PRs and commit history support sole authorship.

**future scope ideas:** enforce the secret via environment variables, add migrations, and keep the testing habit — it is your strongest professional signal.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
