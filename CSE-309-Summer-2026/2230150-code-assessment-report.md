# Code Assessment Report

**Student:** Rabeya Bosri
**ID:** 2230150
**Section:** Section 5
**Project:** Novelia — Online Reading Platform
**Project Type:** Individual
**GitHub:** https://github.com/Rabeya-Bos/Novelia

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`FastAPI()` in `backend/main.py`, ~30 routes)
- Frontend: ✅ React confirmed (Vite project, JSX pages in `frontend/src/view` and `frontend/src/auth`)

## Assessed at Commit
- **SHA:** `df6535f`
- **Date:** 2026-08-13
- **Message:** "Update Novelia project"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 12 |
| First Commit | 2026-06-18 |
| Last Commit | 2026-08-13 |
| Active Days | 5 |
| Contributors | 1 (Rabeya-Bos) |

> Commit history retrieved from the GitHub API (submitted folder contains a snapshot without `.git`). All commits authored by the student alone; development spread from mid-June to mid-August.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| User authentication (register/login) | ✅ Implemented | `POST /api/auth/register`, `/api/auth/login` in `registerbackend.py`/`main.py`; sign-in/register UI wired to backend. |
| Password recovery | ✅ Implemented | PIN-based flow (`forgot-password`, `resend-reset-pin`, `verify-reset-pin`, `reset-password`) with SMTP email delivery + console fallback; 10-minute expiry. |
| User profiles | ✅ Implemented | `GET/PUT /api/profile/{email}` with JSON-file persistence; `UserProfilePage.jsx` wired. |
| Novel discovery / browsing | ⚠️ Partial | `GET /api/novels` exists and `AfterLoginPage.jsx` fetches it, but novels live in a hardcoded in-memory dict (seed data, resets on restart) and `HomePage.jsx` also ships a hardcoded copy of the same array. |
| Genre-based browsing & search/filtering | ⚠️ Partial | Presented via static category sections served from `/api/readerpagebackend` (hardcoded response); no real search/filter endpoints. |
| Personalized recommendations | ⚠️ Partial | `readingHistory` returned by `/api/readerpagebackend` is a hardcoded static list, not per-user data. |
| Comments with likes & ratings | ✅ Implemented | Full comment CRUD + like + rating endpoints; wired in Reader/AfterLogin/Guest pages. In-memory only (lost on restart). |
| Author content publishing | ✅ Implemented | Role-checked author/admin endpoints for series & chapters, persisted to `author_content.json`; wired in `UserProfilePage.jsx`. |
| Admin panel | ⚠️ Partial | `GET /api/adminpanelbackend` is role-checked (admin-only, 403 otherwise) but returns a static role-permission catalog rather than live management operations. |

## Security & Authentication
- Password hashing: ❌ Passwords stored and compared in **plaintext** (`logininfo.json`, direct `user["password"] == payload.password` comparison)
- Token type: ❌ No token/JWT — login returns the user object; session is frontend-only
- Protected routes: ⚠️ Role checks exist on author/admin/comment-delete endpoints (403 with meaningful messages), but they rely on an email/role passed as a query param, and roles are auto-assigned by email domain — anyone can self-register `anything@admin.com` and obtain the admin role
- Role enforcement: ⚠️ Roles (reader/author/admin) exist server-side but the assignment rule (`determine_role_from_email`) is trivially bypassable; novel CRUD endpoints are unprotected
- Password reset: ⚠️ The reset PIN is returned directly in the API response (`"pin": pin`) for any caller, alongside the console fallback

## Data Persistence
- Storage method: ⚠️ Mixed — users and profiles persist to JSON files (`logininfo.json`, written at runtime and re-loaded on startup) and author content to `author_content.json`; novels and comments are **in-memory only** (hardcoded seed / runtime dict, lost on restart)
- Frontend-backend integration: ✅ Wired — `api.js` calls all backend endpoints (auth, profile, novels, comments, author content, admin)

## Runnability
- Backend: ✅ Compiles cleanly; includes an automated test suite (`test_main.py` with `TestClient` covering health, register/login flow, and password reset)
- Frontend: Static review only — Vite React app; pages call `api.js` fetch wrappers
- API wiring: ✅ All pages route data through the backend API

## Observations
- **Good breadth of UI**: six distinct pages (Home, Guest, After-Login, Reader, User Profile, Admin) plus a complete auth flow — impressive for a first full-stack project.
- **Honest report**: the PDF explicitly states the API integration was being added late, which matches the commit history (backend work landed near the end).
- **Security needs urgent fixes**: plaintext passwords, no tokens, and self-registerable admin accounts are the biggest gaps.
- **Persistence is partial**: only users and author content survive restarts; novels and comments are lost.
- **Nice extras**: an automated backend test suite, role-aware 403 responses, SMTP-based password reset with fallback, and full course documentation committed.

## Future Scope
- Hash passwords with bcrypt/passlib and stop returning them anywhere.
- Issue a JWT on login and protect every mutating route with `Depends(get_current_user)`; never trust client-supplied email/role parameters.
- Replace domain-suffix role assignment with explicit role selection (or admin-granted roles) to close the self-register-admin hole.
- Persist novels and comments to JSON files (or migrate to SQLite) so data survives restarts.

## Additional Code-Review Findings

- **The test suite writes to the real user database**: [test_main.py](repo/backend/test_main.py) imports `main`, whose `USERS_FILE` points at the committed `backend/logininfo.json` — every test run registers users into and overwrites the production data file, with no temp-file fixture or teardown. Tests should never mutate live data.
- **The tests codify the vulnerability instead of catching it**: `test_forgot_password_returns_delivery_details_and_pin` explicitly asserts `body['pin']` is present — the automated suite locks in the insecure "PIN in API response" behavior as expected behavior, so a future fix would fail the build.
- **Weak, brute-forceable reset codes**: `build_reset_pin()` uses `random.randint` (a non-cryptographic PRNG — `secrets` is the correct module) to mint a 6-digit PIN, and `is_pin_valid()` performs unlimited comparisons with no attempt counter or lockout, so the ~1M-code space is trivially enumerable against the verify endpoint.
- **The `role` field on `RegisterRequest` is silently ignored**: the request model accepts `role` but `register_user()` discards it in favor of `determine_role_from_email` — dead, misleading API surface that suggests a client can choose a role when it cannot.
- **Registration fabricates profile data**: every new account is created with the hardcoded bio "Curious reader exploring romance…", location "Seattle, USA", and a canned reading goal — invented personal data presented as user-provided.
- **No file locking on the users store**: `save_users_store`/`load_users_store` in [registerbackend.py](repo/backend/registerbackend.py) do read-modify-write on `logininfo.json` with no lock, so two concurrent registrations can silently lose one account.

## Detailed Feedback (Instructor Review)

**What you did well.** You attempted real breadth: six distinct pages, a full register/login flow, PIN-based password recovery with SMTP delivery, profiles, author publishing with role checks, and comment CRUD with likes and ratings. Including an automated `TestClient` suite and honest reporting about late API integration shows self-awareness.

**Where to grow.** The security posture is the weakest part of this submission and needs to be stated plainly: passwords are stored and compared in plaintext, there is no token of any kind, and anyone can self-register `anything@admin.com` to become an admin — the role system is decorative. The reset PIN is even returned in the API response. Persistence is equally incomplete: novels and comments live in memory and vanish on restart, and several features (recommendations, genre browsing, admin panel) are hardcoded static responses, not real functionality. Twelve commits over 5 active days across an 8-week window indicates the bulk of the work landed at the end; the instructor is aware of this gap between the feature list and the sustained effort behind it.

**future scope ideas:** hash passwords, issue JWTs, guard mutating routes, fix role assignment, and persist novels and comments to disk.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
