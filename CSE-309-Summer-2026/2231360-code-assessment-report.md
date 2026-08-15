# Code Assessment Report

**Student:** Tokey Tamim Talha
**ID:** 2231360
**Section:** Section 5
**Project:** Tuition Management Platform
**Project Type:** Individual
**GitHub:** https://github.com/TalhaTamim45/tuition-management-platform

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`main.py` with lifespan seeding; modular routers)
- Frontend: ✅ React confirmed (Vite + React in `frontend/package.json`)

## Assessed at Commit
- **SHA:** `6cfbc96` — 2026-08-13 — "Fix admin quick login password and add admin dashboard API endpoint"
- **Repo:** 21 commits, 7 active days (2026-06-15 → 2026-08-13), single author (TalhaTamim45 / Tamim Talha)

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Role-based auth (client / tutor / admin) | ✅ Implemented | `auth.py` — bcrypt via passlib, PyJWT HS256 tokens (24h exp), `get_current_user`, `require_admin` / `require_tutor` / `require_client` dependency factories (403 on mismatch), **blocked-account enforcement** (403 "suspended by an administrator"). |
| Tuition post CRUD (clients) | ✅ Implemented | `routes_tuition_posts.py` (241 lines, 12 Depends, 6 role checks) — publish/edit/delete own posts; owner-or-admin authorization on update/delete. |
| Tutor applications + duplicate prevention | ✅ Implemented | `routes_applications.py` — apply to posts, status tracking; backend rejects duplicate applications with 400 + frontend `appliedPostIds` state disables the button (race-condition fix documented in report). |
| Client accept/reject applications | ✅ Implemented | Application status updates with tutor academic detail review; `MyApplications.jsx` for tutors. |
| Tutor profiles (academic details) | ✅ Implemented | `models.py` User table stores education, institution, subjects, experience, salary expectation, address; `UserProfile.jsx`. |
| Admin moderation | ✅ Implemented | `routes_admin.py` (168 lines, 8 `require_admin`) — user management, block/unblock (enforced in `get_current_user`), post removal, admin dashboard API. |
| Admin dashboard UI | ✅ Implemented | `AdminDashboard.jsx`, `AdminUsers.jsx`, `AdminTuitionPosts.jsx`. |
| Seeded demo environment | ✅ Implemented | Lifespan context seeds admin + client + tutor + sample post/application (`main.py`). |

## Security & Authentication
- Password hashing: ✅ bcrypt (`passlib`) — *with a documented SHA-256 fallback if bcrypt fails* (⚠️ unsalted fallback — flagged below)
- Token type: ✅ Real JWT (PyJWT, HS256, `exp`, 24h)
- Protected routes: ✅ `HTTPBearer` + `get_current_user` on all data-touching routes (12 Depends in tuition posts, 8 require_admin in admin routes)
- RBAC: ✅ `require_admin/require_tutor/require_client` with 403s; ownership checks (owner-or-admin) on post mutation
- Blocked accounts: ✅ rejected at auth level
- ⚠️ Weaknesses to note:
  1. `auth.py` hardcodes a **fallback `SECRET_KEY`** when env is unset — tokens forgeable on misconfigured deployments
  2. SHA-256 fallback hashing is unsalted if bcrypt errors (bcrypt/passlib version conflicts are common)
  3. CORS `allow_origins=["*"]` with `allow_credentials=True` (browsers reject/warn; should enumerate origins)
  4. Seeded default admin password (`admin123` / demo creds) — fine for demo, must change in production

## Data Persistence
- Storage: ✅ **SQLite** via SQLAlchemy (`tuition_platform.db`); 3 tables: User, TuitionPost, Application; `Base.metadata.create_all` on startup lifespan
- Frontend wiring: ✅ All components call backend via native `fetch` with JWT Bearer headers; no hardcoded data arrays as persistence
- No Alembic — schema changes handled by teardown/recreate during dev (documented in report)

## Runnability & Tests
- Backend: ✅ All 11 Python files pass `py_compile`
- Tests: ✅ 8 pytest tests across `test_admin_and_roles.py` (131 lines) and `test_tuition_posts.py` (137 lines) — auth, role enforcement, unauthorized access, CRUD flows via TestClient
- Frontend: static review only (Vite build config present)
- Helper: `verify.ps1` for environment verification

## Observations
- **Textbook FastAPI pattern**: lifespan seeding, dependency-injected role guards, Pydantic schema validation, ownership checks — clean solo implementation.
- **Honest engineering reflection**: the duplicate-application race fix (backend 400 + frontend disabled state) and schema-migration teardown approach show real debugging maturity.
- Docs: 22 files (PRD, SRS with DFD, TDD with ERD, API design, etc.).
- All code single-author; commit messages descriptive ("Fix admin quick login password and add admin dashboard API endpoint").

## Future Scope
- Make `SECRET_KEY` mandatory (fail fast if unset) instead of falling back to a hardcoded default.
- Remove the SHA-256 fallback hash path or salt it; pin `bcrypt==4.0.1` if passlib compatibility is the concern.
- Restrict CORS origins to the actual frontend URL(s) and drop wildcard + credentials.
- Add Alembic for schema migrations instead of teardown/recreate.

## Additional Code-Review Findings

- **Duplicate-application prevention is not race-safe**: `apply_for_tuition_post` (`backend/routes_tuition_posts.py`, line 141) does a SELECT-then-INSERT with no `UniqueConstraint(tuition_post_id, tutor_id)` on the `Application` model (`backend/models.py`). Two concurrent requests can both pass the "already applied" check and insert duplicates. The backend 400 handles the sequential case, but the database itself must enforce uniqueness.
- **No password policy at registration** (`backend/routes_auth.py`): `register` accepts any non-empty password — a one-character password is valid. There is no minimum length or complexity rule anywhere in `schemas.py` or the route.
- **Registration is a user-enumeration oracle**: `POST /api/auth/register` returns `400 "User with this email already exists"`, while login correctly uses the generic "Invalid email or password". An attacker can harvest registered emails via the register endpoint.
- **Application status transitions are unguarded** (`backend/routes_applications.py`): a client can move an application from `accepted` back to `pending` or flip `rejected` → `accepted` arbitrarily — there is no state machine restricting which transitions are legal, so history can be rewritten after a decision was communicated.
- **N+1 query pattern in `get_my_applications`** (`backend/routes_applications.py`): the loop accesses `app.tuition_post` lazily per application, issuing one extra query per row; use a joinedload/selectinload or a single join instead.
- **Positive: tests use proper DB isolation**: both test files override `get_db` with a separate SQLite engine and rebuild the schema per run via the autouse fixture (`backend/test_admin_and_roles.py`) — a genuinely sound pattern. However, tests never set `SECRET_KEY`, so they implicitly exercise the hardcoded fallback key and would not catch a misconfigured deployment.

## Detailed Feedback (Instructor Review)

**What you did well**
A clean, textbook FastAPI implementation: bcrypt via passlib, PyJWT tokens, `HTTPBearer` with `get_current_user`, `require_admin`/`require_tutor`/`require_client` factories raising 403, owner-or-admin authorization on post mutations, and blocked-account enforcement at the auth layer. You shipped 8 pytest tests covering auth, role enforcement, and CRUD flows — real verification most submissions lack. The duplicate-application race fix (backend 400 plus frontend disabled state) shows honest, mature debugging, and the lifespan seeding gives reviewers a working demo out of the box.

**Where to grow**
Four concrete security issues need fixing: the hardcoded `SECRET_KEY` fallback makes tokens forgeable on a misconfigured deployment; the unsalted SHA-256 fallback hash path should be removed or salted; CORS wildcard with credentials must become an enumerated origin list; and the seeded demo admin password must never reach production. SQLite with `create_all` and teardown/recreate is acceptable at this scale but will not survive real schema evolution.

future scope ideas: fail fast on a missing secret, drop the fallback hash, restrict CORS origins, and adopt Alembic migrations before your next schema change.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
