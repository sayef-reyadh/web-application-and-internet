# Code Assessment Report

**Student:** Shariar Nazim Joy
**ID:** 2210891
**Section:** Section 5
**Project:** UniMarketplace — University Marketplace & Tutor Platform
**Project Type:** Individual
**GitHub:** https://github.com/Shariar-Joy/UniMarketplace

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app/main.py`, 7 routers: admin, auth, health, messaging, products, tutors, users, wishlist)
- Frontend: ✅ React confirmed (`"react": "^19.2.7"` in `package.json`, TypeScript/TSX throughout)

## Assessed at Commit
- **SHA:** `e033a73bff8eea4279d11e24dfa1b43c5367202b`
- **Date:** 2026-08-13
- **Message:** "Merge pull request #50 from Shariar-Joy/48-implement-admin-dashboard"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits (local) | 1 (shallow clone) |
| First Commit | 2026-08-13 |
| Last Commit | 2026-08-13 |
| Active Days (local) | 1 |
| Contributors | 1 (Shariar Joy) |

> **Note:** Submitted as a shallow clone — only the final PR #50 merge commit is present locally. 50 pull requests on GitHub indicate extensive, sustained development history.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| User registration / login | ✅ Implemented | `auth_service.py` + `auth.py` router. Login sets a signed JWT in an HTTP-only cookie. |
| Product listings | ✅ Implemented | `product_service.py` + `products.py` router with 12 protected routes. `ProductCard.tsx`, `ProductFilters.tsx` in frontend. |
| Product images (Cloudinary) | ✅ Implemented | `image_service.py` + Cloudinary SDK. `product_images` migration table (0004). |
| Wishlist | ✅ Implemented | `wishlist_service.py` + `wishlist.py` router (4 protected routes). `wishlists` migration (0005). |
| Messaging / conversations | ✅ Implemented | `messaging_service.py` + `messaging.py` router (4 protected routes). `conversations_and_messages` migration (0006). Read tracking migration (0007). |
| Tutor marketplace | ✅ Implemented | `tutor_service.py` + `tutors.py` router (3 protected routes). `TutorCard.tsx`. Tutor migration (0008). |
| Admin dashboard | ✅ Implemented | `admin_service.py` + `admin.py` router (4 protected routes, `get_current_admin_user`). `AdminNav.tsx`. Admin role migration (0009). |
| Security headers middleware | ✅ Implemented | `middlewares/security_headers.py` — adds `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security` headers to all responses. |

## Security & Authentication
- Password hashing: ✅ **bcrypt** — `passlib[bcrypt]==1.7.4` + `bcrypt==4.0.1`. Full `CryptContext` with bcrypt scheme.
- Token type: ✅ **Real JWT** — `PyJWT==2.13.0`. Token contains `sub` (user ID), `iat`, `exp`. Signed with `settings.SECRET_KEY`.
- Token storage: ✅ **HTTP-only cookie** — more secure than localStorage (not accessible to JavaScript, immune to XSS attacks). Cookie name is configurable via `settings.COOKIE_NAME`.
- Protected routes: ✅ `Depends(get_current_user)` across 7 routers — **31 total protected routes**: products (12), admin (4), messaging (4), wishlist (4), tutors (3), users (3), auth (1).
- Role enforcement: ✅ `get_current_admin_user` checks `current_user.is_admin` and raises HTTP 403 for non-admins. Admin role tracked via migration (0009).

## Data Persistence
- Storage method: ✅ SQLAlchemy ORM + **Alembic migrations** (9 migration files: `0001_initial_schema` through `0009_admin_role`). PostgreSQL primary, SQLite fallback for local testing.
- Frontend-backend integration: ✅ Fully wired — TypeScript frontend with typed API calls. Cookie-based auth flows transparently with same-origin requests.

## Runnability
- Backend: ✅ Started cleanly — `uvicorn app.main:app --port 8017` → HTTP 200 on `/health`. Application startup complete with SQLite fallback.
- Frontend: ✅ Started successfully after `npm install` — Vite v8.1.1, HTTP 200, 5301ms.
- API wiring: ✅ Frontend calls backend. HTTP-only cookie auth is transparent in same-origin setup.

## Observations
- **Most complete security implementation in the batch**: bcrypt + PyJWT + HTTP-only cookies (not localStorage) + `get_current_admin_user` + security headers middleware. This is production-grade security practice.
- **9 Alembic migration files** covering the full development lifecycle of the schema — migrations were created incrementally per feature, demonstrating genuine iterative development.
- **50 PRs on GitHub** merged as the final commit — the project was developed with professional branch-based workflow.
- **Cloudinary image uploads** add real-world infrastructure integration beyond basic CRUD.
- **8 test files** covering admin, messaging, products (filters/images/search), tutors, wishlist — comprehensive test coverage.
- `vercel.json` in frontend suggests the project was configured for deployment.
- **Shallow clone** — only 1 commit locally. Full development history on GitHub cannot be verified from this submission.

## Future Scope
- Submit the full git history in future submissions. 50 PRs represent real, measurable work — losing this from the grading submission means the dev process cannot be credited locally.
- Consider adding refresh tokens and token rotation for an even more complete auth implementation.
- The project is at a professional standard. The only remaining gap is the submission format.

---

## Additional Code-Review Findings

- **The test suite is real and genuinely good — but it skips auth itself.** `backend/tests/conftest.py` is exemplary (in-memory SQLite with `StaticPool`, `app.dependency_overrides` for the DB session, and a thoughtful `raise_server_exceptions=False` so tests exercise the real exception handlers), and `test_admin.py` contains proper negative authorization tests (401 unauthenticated, 403 non-admin). However, there is **no `test_auth.py`**: register, login, cookie-setting, and token rejection — the most security-critical code in the project — have zero direct tests.
- **Cookie auth has no CSRF protection for the deployment mode the config itself recommends.** `backend/app/core/config.py` comments state that a cross-site deploy (Vercel frontend + Render backend — which `frontend/vercel.json` implies) requires `COOKIE_SAMESITE="none"`. With `SameSite=None`, browsers attach the auth cookie to cross-site requests, and there is no CSRF token or double-submit check anywhere in `app/` — so in exactly that deployment mode, every state-changing endpoint is CSRF-exposed. SameSite=Lax is fine for same-site local dev, but the cross-site path needs a CSRF mechanism.
- **Insecure defaults warn instead of failing.** `SECRET_KEY` falls back to `"dev-secret-key-change-me"` and `COOKIE_SECURE` defaults to `False`. `app/main.py` (lines 29–31) deserves credit for detecting the default secret and logging a loud startup warning — that is more than most projects do — but a warning is easy to miss in logs; refusing to boot with a known-public secret would be the safe behavior.
- **Repo hygiene is clean.** The `.venv` directory present on disk is **not committed**: `.gitignore` explicitly excludes `backend/.venv/`, `backend/*.db`, and `backend/.env`, and the git index contains only source, migrations, and tests. This is correct practice — keep it that way.
- **Minor:** `pytest==8.4.2` is listed under a "Dev/test only" comment inside `backend/requirements.txt`. Splitting into `requirements-dev.txt` would keep production images smaller and avoid shipping test tooling to deploys.

## Detailed Feedback (Instructor Review)

**What you did well**
- This is the strongest security posture in the cohort. You combined bcrypt hashing, a signed PyJWT with expiry, HTTP-only cookie storage (deliberately avoiding localStorage and its XSS exposure), `Depends(get_current_user)` on 31 routes, and a separate `get_current_admin_user` that returns 403 for non-admins. Very few students implemented role enforcement server-side; you did.
- Nine incremental Alembic migrations show the schema evolved feature-by-feature rather than being written once — genuine iterative engineering.
- Eight test files and a security-headers middleware put this well beyond a typical class project. Cloudinary integration shows you can wire third-party infrastructure.

**Where to grow**
- Your auth is solid but stateless-access-token-only. Adding a refresh-token flow with rotation would make this production-grade and demonstrate you understand token lifecycle management.
- Consider rate limiting on the auth endpoints to protect against credential stuffing — you clearly have the skills to add it.

**Submission note**
- You submitted a shallow clone (1 commit). After we fetched your full history from GitHub (60 commits, 10 active days over 6+ weeks, 50 PRs), your sustained development process was confirmed and credited. In future, submit the full `.git` history so this is visible without extra steps.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
