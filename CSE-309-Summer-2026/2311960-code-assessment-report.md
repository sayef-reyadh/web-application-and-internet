# Code Assessment Report

**Student:** Md. Mahmudul Hasan
**ID:** 2311960
**Section:** Section 6
**Project:** VERA — Volunteer Emergency Response Alliance
**Project Type:** Team (with Ridwan Hasan Khandakar 2310604)
**GitHub:** https://github.com/MahmudulHasanJoy/VERA

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/`, 8 route modules)
- Frontend: ✅ Next.js (React) confirmed (`frontend/`, 18 pages) — Next.js counts as React

## Assessed at Commit
- **SHA:** `553e922` — 2026-08-13 — "Fix in-app alerts for blood and emergencies with unread badge UI"
- **Repo:** 58 commits, 12 active days (2026-06-13 → 2026-08-13)
- **This student:** 53 commits (`Mahmud2311960` 33 + `Mahmudul Hasan Joy` 20), 12 days
- **Teammate (Ridwan):** 5 commits, 2 days — SRS + TDD docs only (see below)

## Individual Contribution — verified per-path authorship
Nearly every file is Mahmud's (GitHub API per-path scan):
- Backend: `main.py` 4/4, `core/config.py` 6/6, `core/security.py` 1/1, `api/deps.py` 1/1, all 8 route modules (auth 3, emergencies 2, blood 2, features 2, assistant 1, search 1, reports 1, stats 1), `services/notifications.py` 5/5, `services/assistant.py` 1/1, controllers, repositories
- Frontend: `lib/api.ts` 7/7, dashboard/admin/emergencies/blood pages, `VeraAssistant.tsx`
- Tests: `tests/test_api.py` 4/4, `tests/conftest.py` 3/3
- Docs: business-analysis, SRS, TDD folders
- **Teammate Ridwan's 5 commits** are confined to the root `SRS/` and `TDD/` doc folders (4 commits each file set) — no code.

## Features Found
- Auth: register/login/me, **forgot/reset password with email + expiry**, 6 roles (citizen, volunteer, donor, NGO, hospital, admin)
- Emergencies (create/list/manage), blood donor matching, shelters, incidents, resources, donations + campaigns, volunteers + certificates, disaster coverage, location-aware search, in-app notifications with unread badge
- **Agentic AI assistant (VERA Bot)** — Gemini-powered, queries live platform data via tools (`services/assistant.py` + `assistant_tools.py`)
- Frontend: 18 pages, AuthGuard, Supabase session handling, admin panel, dashboard

## Security & Authentication (verified)
- ✅ passlib **bcrypt** hashing
- ✅ Real JWT (python-jose, HS256, exp) via `core/security.py`
- ✅ `OAuth2PasswordBearer` + `get_current_user` (DB re-query + `is_active` check)
- ✅ **`require_roles(*roles)` dependency factory** → 403 for wrong roles (admin auto-bypass)
- ✅ Password-reset tokens with expiry; `.env.example`; `SECURITY_REVIEW.md` doc
- ⚠️ `secret_key` default `"dev-secret-change-in-production"` (env-overridable via pydantic-settings)

## Data Persistence
- ✅ **PostgreSQL (Supabase)** via SQLAlchemy + **alembic migrations** (0001_initial, 0002_password_reset), docker-compose (prod), Railway + Vercel live deploys

## Runnability (tested in this session)
- ✅ **9/9 backend tests pass** (auth, duplicate email, invalid login, /me with token, emergency CRUD; needed `bcrypt<4.1` due to known passlib incompatibility, Python 3.11+ required for `datetime.UTC`)
- ✅ Live: frontend on Vercel, API on Railway, Supabase PostgreSQL

## Observations
- Production-grade team project carried essentially single-handedly: 8 API modules, AI assistant, alembic migrations, 9 passing tests, deploy configs (Railway, Vercel, OCI), plus QA docs (REGRESSION_PASS, PERFORMANCE_SMOKE, SECURITY_REVIEW, RELEASE_NOTES).
- Teammate's contribution is documentation-only (SRS/TDD); Mahmud's 16-page PDF report is detailed and honest about this split.

## Future Scope
- Raise the passlib/bcrypt pin in `requirements.txt`; remove the hardcoded dev secret default (require env in production); get teammate code contributions into the repo.

---

## Additional Code-Review Findings

- **`RESET_DB=1` is a production kill-switch.** The lifespan handler in `backend/app/main.py` runs `Base.metadata.drop_all(bind=engine)` whenever the `RESET_DB` environment variable is `1` — if that variable is ever set on your Railway deployment, a routine restart wipes the entire production database.
- **Password-reset token leaked in the API response.** When SMTP is not configured, `POST /auth/forgot-password` (`backend/app/api/routes/auth.py`) returns the raw `reset_token` and full `reset_url` in the HTTP response — anyone who knows a victim's email can reset their password without mailbox access. The differing response shape also quietly confirms which emails are registered, defeating the generic-message anti-enumeration intent.
- **No token revocation.** JWTs are stateless with no logout endpoint and no invalidation: `reset-password` changes the password but previously issued access tokens remain valid until their 24-hour expiry.
- **The AI assistant is anonymously accessible.** `/assistant/chat` uses `auto_error=False` optional auth, so unauthenticated callers can drive the Gemini integration — your API quota and cost are exposed to the open internet with no rate limiting.
- **Dual schema management.** Startup runs `Base.metadata.create_all` alongside your alembic migrations — the two can silently drift, letting the app boot against a schema your migrations never produced.
- **Test coverage is concentrated in two modules.** `backend/tests/test_api.py` exercises auth and emergencies only; six of your eight route modules (blood, features, assistant, search, reports, stats) have no tests at all.

---

## Detailed Feedback (Instructor Review)

**What you did well.** You carried this team project, and the git record proves it. Real JWT authentication (python-jose, HS256 with expiry), passlib bcrypt hashing, and a `require_roles` dependency factory that correctly returns 403 for wrong roles — this is how auth should be done at this level. You built essentially the entire platform: all backend route modules, password reset with expiring email tokens, alembic migrations against PostgreSQL, an agentic AI assistant that queries live platform data through tools, the Next.js frontend, a passing test suite, and live Railway/Vercel deploys. The QA documents (regression pass, performance smoke, security review) show professional discipline.

**Where to grow.** A hardcoded `dev-secret-change-in-production` default has no place in a deployed app — fail fast at startup instead. Your passlib/bcrypt version pin is fragile and cost setup time when running your tests. And a team project where one member writes all the code is a process failure you should surface and manage, not absorb silently.

**Attribution note.** Per-path authorship scanning confirms nearly every code file is yours; Ridwan's five commits are confined to SRS/TDD documentation. Your assessment reflects that reality.

**future scope ideas:** remove the secret default, fix the dependency pin, and split ownership visibly in your next team project.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
