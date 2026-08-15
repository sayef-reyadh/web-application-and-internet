# Code Assessment Report

**Student:** Solaiman Munna
**ID:** 2111315
**Section:** Section 5
**Project:** CoursePilot — Academic Course Registration System
**Project Type:** Individual
**GitHub:** https://github.com/SS-Munna/CoursePilot

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app/main.py`, 10 routers registered)
- Frontend: ✅ React confirmed (`"react": "^19.2.7"` in `package.json`, TypeScript/TSX throughout)

## Assessed at Commit
- **SHA:** `e4a3104cd4e4a552a0b2e504a3e53f97a8fd5e1a`
- **Date:** 2026-08-13
- **Message:** "Merge pull request #87 from SS-Munna/munna-issue#42-final-security-regression"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits (local) | 1 (shallow clone) |
| First Commit | 2026-08-13 |
| Last Commit | 2026-08-13 |
| Active Days (local) | 1 |
| Contributors | 1 (Solaiman Munna) |

> **Note:** The submitted repository contains only 1 commit — a final merge of PR #87. The commit message references 87 pull requests on GitHub, indicating substantial development history exists on the remote. However, only the merged snapshot was submitted locally, so active-day history cannot be verified from this submission.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| User authentication (login/logout/me) | ✅ Implemented | `auth.py` router. Real JWT issued on login, `/api/auth/me` returns current user via `Depends(get_current_user)`. |
| 4-role access control (Student, Advisor, Dept Admin, System Admin) | ✅ Implemented | `UserRole` enum in `authorization.py`. `require_roles()` factory used throughout all 10 routers. |
| Course management | ✅ Implemented | `courses.py` router — create, list, detail, update courses. Prerequisites modeled in `course_prerequisite.py`. |
| Course selection / pre-registration | ✅ Implemented | `selections.py` router with 14 auth-protected lines. `selectionApi.ts` in frontend. |
| Registration periods | ✅ Implemented | `registration_periods.py` router. Period open/close logic with `registration_period_status_repository.py`. |
| Final registration submission | ✅ Implemented | `registrations.py` router + `registration_submission_repository.py`. |
| Waitlist system | ✅ Implemented | `waitlists.py` router + `waitlist_promotion_repository.py`. Automatic waitlist promotion on seat release. |
| Advisor reviews | ✅ Implemented | `advisor_reviews.py` router + `advisorApi.ts` frontend. |
| Notifications | ✅ Implemented | `notifications.py` router (3 protected routes) + `notificationApi.ts` frontend service. |
| Audit logs | ✅ Implemented | `audit_logs.py` router + `auditApi.ts`. Admin can view system audit trail. |
| Admin panel | ✅ Implemented | `admin.py` router (20 auth-protected lines) — users, departments, programs, staff management. `adminApi.ts` in frontend. |
| Credit validation / schedule conflict detection | ✅ Implemented | Dedicated repositories: `credit_repository.py`, `schedule_conflict_repository.py`, `prerequisite_repository.py`. |

## Security & Authentication
- Password hashing: ⚠️ SHA-256 via Python `hashlib.sha256` — not bcrypt. SHA-256 is a general-purpose hash (very fast), making it vulnerable to brute-force and rainbow table attacks. Not a recommended password hashing algorithm.
- Token type: ✅ Real JWT — PyJWT 2.13.0 signs tokens with HMAC using `settings.jwt_secret_key`. Payload includes `sub`, `iat`, `exp` (30-minute expiry).
- Protected routes: ✅ `Depends(get_current_user)` and `Depends(require_roles(...))` used across all 10 routers. 401 returned for missing/invalid tokens, 403 for role violations.
- Role enforcement: ✅ `require_roles(UserRole.SYSTEM_ADMIN)` guards admin routes. `ensure_owner_or_roles()` used for ownership checks. `forbidden_exception()` returns HTTP 403 with structured error codes.

## Data Persistence
- Storage method: ✅ SQLAlchemy ORM with PostgreSQL (psycopg2-binary). SQLite fallback works for local testing. `schema_migrations.py` runs on startup. 17 repositories covering every data operation.
- Frontend-backend integration: ✅ Fully wired — 10 dedicated API service files (`adminApi.ts`, `authApi.ts`, `courseApi.ts`, `selectionApi.ts`, `waitlistApi.ts`, etc.) all calling the backend.

## Runnability
- Backend: ✅ Started successfully — `uvicorn app.main:app --port 8013` → HTTP 200 on `/docs`. Started cleanly with SQLite fallback.
- Frontend: ✅ Started successfully after `npm install` — Vite v8.1.2, HTTP 200, 1633ms.
- API wiring: ✅ Frontend has 10 service files calling `/api/*` endpoints with Bearer token authentication.

## Observations
- **Most technically sophisticated project assessed so far**: layered architecture (models → repositories → schemas → API routes → authorization), schema migrations, comprehensive test suite (25+ test files covering JWT security, prerequisites, waitlists, schedule conflicts, seat allocation, API regression).
- **4-role RBAC with a factory pattern** (`require_roles(*allowed_roles)`) and ownership checks is production-grade design.
- **Password hashing is the one notable gap**: replacing `hashlib.sha256` with `passlib[bcrypt]` is a one-line change that would make the security layer fully professional.
- **Shallow clone submission**: only 1 commit exists in the submitted repo. The full development history (87+ PRs) remains on GitHub but is not verifiable from the submission.
- `node_modules` not committed — correct practice, but required `npm install` before running.
- TESTING.md and README.md (27KB) are detailed and professional.

## Future Scope
- Replace `hashlib.sha256` with `passlib[bcrypt]` for password hashing — this is the single most impactful security improvement remaining.
- For future submissions, ensure the submitted repository contains the full commit history (e.g., `git clone --mirror` or push the development branch directly rather than a single squash-merge).
- The project is otherwise at an exceptional level of implementation depth.

## Additional Code-Review Findings

- **The frontend is tested too — this deserves explicit credit.** There are 19 Vitest files under `frontend/src/test/` covering pages (`LoginPage.test.tsx`, both dashboards), role-based routing (`AppRoleRouting.test.tsx`), and one test per API service module (`adminApi.test.ts`, `waitlistApi.test.ts`, etc.). Combined with the 28 backend `test_*.py` files, that is 47 test files — the strongest coverage in the section.
- **Timing-safe comparison done right:** `verify_password` in `backend/app/security.py` uses `hmac.compare_digest`, so hash comparison is not vulnerable to timing attacks. The algorithm choice (SHA-256) is the flaw, but the comparison itself was implemented correctly.
- **Strict JWT claim validation:** `decode_access_token` enforces `options={"require": ["sub", "iat", "exp"]}`, so malformed or claim-deficient tokens are rejected rather than silently accepted — a detail many implementations miss.
- **Hardcoded fallback JWT secret:** `backend/app/config.py` defaults `jwt_secret_key` to `"development-only-change-this-secret"`. If `JWT_SECRET_KEY` is unset, tokens are signed with a publicly known key. Fail fast when the secret is missing outside development instead of defaulting.
- **No hardcoded admin credentials:** bootstrap admin provisioning reads `BOOTSTRAP_SYSTEM_ADMIN_*` env vars that default to empty strings — a clean, correct pattern with no secrets baked into source.
- **Repo hygiene verified:** only `.env.example` files are tracked — no `.env`, virtualenv, or database files are committed.

## Detailed Feedback (Instructor Review)

**What you did well:** This is the most technically sophisticated project I assessed. A proper layered architecture — models, repositories, schemas, routes, authorization — with 17 repositories, schema migrations that run on startup, and a test suite of 25+ files covering JWT security, waitlists, schedule conflicts, and seat allocation. Your 4-role RBAC built on a `require_roles()` factory, ownership checks via `ensure_owner_or_roles()`, and structured 401/403 responses are production-grade design. The frontend's ten dedicated API service files show disciplined integration, and the 27KB README plus TESTING.md are genuinely professional.

**Where to grow:** One real flaw: passwords are hashed with unsalted SHA-256 via `hashlib`. SHA-256 is designed to be fast — exactly what you don't want for passwords — and without a salt it's exposed to rainbow tables. For someone working at this level, this is a surprising oversight. Swapping in bcrypt is nearly a one-line change.

**Submission note:** You submitted a shallow clone containing a single squash-merge commit. Your commit messages reference 87 pull requests, so real history clearly exists on GitHub — but I can only assess what was submitted, and your development journey is not verifiable from this snapshot. Submit full history next time.

**future scope ideas:** Fix the password hashing, and submit complete git history.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
