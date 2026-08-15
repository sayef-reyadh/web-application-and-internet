# Code Assessment Report

**Student:** Asif
**ID:** 2330522
**Section:** Section 6 (Team 3)
**Project:** Student Study Planner App
**Project Type:** Individual (per student_map; repo shows a collaborator `AfifaHaque` — see Observations)
**GitHub:** https://github.com/asifahmed6969/WebApp-1

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend-fastapi/app/`, 5 routers + controllers + models + dependencies)
- Frontend: ✅ React confirmed (`frontend/`, 10 pages)

## Assessed at Commit
- **SHA:** `7c2a5db` — 2026-08-09 — "Add role-based admin panel and user management"
- **Repo:** 73 commits, 15 active days (2026-06-17 → 2026-08-09)
- **This student:** 33 commits (`asifahmed6969`), **10 distinct days** (06-17 → 08-09)

## Individual Contribution — verified per-path authorship
Repo is co-authored with `AfifaHaque` (40 commits). Per-path authorship shows:

| Area | Owner |
|------|-------|
| `main.py` (6/7), `database.py` | Asif (app bootstrap, CORS, MongoDB wiring) |
| Admin module (routes + controller + AdminPanel + AdminRoute) | ✅ Asif 100% (role management, user delete, self-deletion/demotion protection) |
| Schedule + Material modules (routes + controllers + pages) | ✅ Asif 100% |
| Calendar, DeadlineTracker, FileManager, ProgressTracking, Register pages | ✅ Asif (3 commits each) |
| AuthContext, api.js (Axios interceptor), ProtectedRoute | ✅ Asif |
| Auth controller (bcrypt + JWT), `dependencies/auth.py` | Shared (1+1 each) |
| Task module (routes/controller/models + TaskManagement page), Login, overview docs | AfifaHaque (teammate) |

His report claims the whole project as individual work — accurate for his modules, but `AfifaHaque` (40 commits) contributed the task module, login, and docs. Not a deduction: his claims match his own commits; he is the dominant author of backend infrastructure, auth, RBAC, and 6 of 9 feature pages.

## Features Found
- JWT register/login (students forced to `role=student` at registration — privilege-escalation fix documented)
- Task CRUD + search/filter + mark-complete, all user-scoped (`user_id` in every query)
- Study schedule, study materials, calendar, deadline tracker, progress tracking, profile — all wired to authenticated user data
- Admin panel: list users, promote/demote, delete (with self-deletion and self-demotion blocked in both frontend and backend)

## Security & Authentication (verified)
- ✅ Native **bcrypt** hashing with per-user salt
- ✅ Real PyJWT (HS256, exp, 60-min), HTTPBearer
- ✅ `get_current_user` re-queries MongoDB for the latest user record (not just token claims)
- ✅ `require_admin` on all 3 admin routes → **403** for non-admins; user-scoped data access
- ⚠️ `JWT_SECRET` hardcoded fallback in `dependencies/auth.py` and `auth_controller.py` (env override available)
- CORS restricted to local dev origins + `https://*.vercel.app` regex

## Data Persistence
- ✅ **MongoDB Atlas** (cloud NoSQL) via PyMongo with env-based connection string (`MONGO_URI`, `DATABASE_NAME`), collections for users/tasks/schedules/materials, explicit connection check endpoint

## Runnability
- ✅ All 22 backend Python files pass `py_compile`
- ✅ Deployed: frontend on Vercel, backend on Render (auto-redeploy from main)
- Runtime DB access requires valid Atlas credentials (env); config is correct

## Observations
- Genuinely complete full-stack app with the strongest security setup seen so far in S6: bcrypt + JWT + DB-refreshed current user + enforced RBAC + user-scoped queries + forced-student registration.
- Solid documentation (17 docs) and deployment discipline (Render + Vercel, `.gitignore` for env).
- Minor: `lost-and-found/` template folder leftover in repo root (unrelated app stub); hardcoded `JWT_SECRET` fallback.

## Future Scope
- Remove the stray `lost-and-found/` folder; make `JWT_SECRET` env-required (no fallback); document the AfifaHaque collaboration in the report.

## Additional Code-Review Findings

- The JWT is stored in `localStorage` (see `frontend/src/context/AuthContext.jsx` and the reads in `frontend/src/services/api.js`), which makes the token readable by any JavaScript running on the page. Combined with the absence of any server-side logout/token-invalidation mechanism, a single XSS payload would yield a token that stays usable until its 60-minute expiry. Short-lived tokens plus an httpOnly cookie (or an explicit revocation strategy) would reduce this exposure.
- Eight of the ten pages import raw `axios` and hand-roll their own `getAuthHeader()` (e.g. `frontend/src/pages/FileManager.jsx`, `CalendarView.jsx`, `DeadlineTracker.jsx`, `StudySchedule.jsx`, `TaskManagement.jsx`) instead of using the centralized `services/api.js` instance whose interceptor already attaches the token. This duplicates auth logic across the codebase and means a future change to the auth scheme must be edited in many places; it also invites the inconsistencies already visible (some pages `alert()` on 401, others fail silently).
- The CORS `allow_origin_regex=r"https://.*\.vercel\.app"` in `backend-fastapi/app/main.py` trusts **every** Vercel-hosted site, not just your deployment — any unrelated (or malicious) `*.vercel.app` app can make credentialed cross-origin requests to this API. Pin the origin to your exact frontend domain.
- Registration validation in `backend-fastapi/app/models/user.py` enforces only `min_length=6` on passwords, and `backend-fastapi/app/routes/auth_routes.py` exposes `/api/auth/login` with no rate limiting or lockout, leaving the login endpoint open to unrestricted credential guessing against otherwise-solid bcrypt hashes.
- The study-material `url` field is stored unvalidated and rendered directly as `<a href={material.url}>` in `frontend/src/pages/FileManager.jsx`; React does not sanitize `href`, so a `javascript:` URL submitted via a direct API call (bypassing the `type="url"` input, which is client-side only) would execute on click. Validate/scheme-allowlist URLs server-side in `material_controller.py`.
- The schedule and material routers (`backend-fastapi/app/routes/schedule_routes.py`, `material_routes.py`) expose only list/create/delete — there is no update endpoint, so editing any entry requires delete-and-recreate, which loses the original `createdAt` ordering.

## Detailed Feedback (Instructor Review)

**What you did well.** A genuinely complete full-stack application with the strongest security posture in your section: native bcrypt hashing, real PyJWT tokens with expiry, `require_admin` returning 403 on every admin route, database-refreshed user lookups rather than trusting token claims, user-scoped queries throughout, and forced-student registration that closes a privilege-escalation hole. MongoDB Atlas persistence, a working admin panel with self-demotion protection, and 33 well-spread commits show consistent, disciplined work.

**Where to grow.** Two fixable weaknesses: the hardcoded `JWT_SECRET` fallback in the auth code — make the environment variable mandatory so a missing secret fails loudly — and the stray `lost-and-found/` template folder in the repo root, which suggests untidied scaffolding. Test coverage is absent; an application this complete deserves at least API-level tests for the auth and admin paths.

**Attribution note.** Submitted as individual work, but the repository shows a collaborator with 40 commits owning the task module and login page. Your claims match your own commits, so there is no misconduct — however, your report should have disclosed this collaboration explicitly.

**future scope ideas:** remove the dead folder, require `JWT_SECRET` from the environment, add auth/admin tests, and document the collaboration.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
