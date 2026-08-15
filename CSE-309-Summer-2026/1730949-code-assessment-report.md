# Code Assessment Report

**Student:** Syeda Afifa Siddiqua
**ID:** 1730949
**Section:** Section 5
**Project:** FoodBridge — Food Sharing & Donation Platform
**Project Type:** Individual
**GitHub:** https://github.com/Syeda-Afifa/FoodBridge

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/main.py`, 4 routers registered)
- Frontend: ✅ React confirmed (`"react": "^19.1.0"` in `package.json`, TypeScript/TSX components)

## Assessed at Commit
- **SHA:** `67eb8cb2b680001c1946047fed14efaa978ec275`
- **Date:** 2026-08-13
- **Message:** "Restrict food request submission to recipients"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 45 |
| First Commit | 2026-06-13 |
| Last Commit | 2026-08-13 |
| Active Days | 6 |
| Contributors | 1 (Syeda-Afifa) |

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| User registration & login (Donor/Recipient/Admin) | ✅ Implemented | `app/controllers/auth_controller.py` — signup/login/logout/refresh. Role-based registration enforced. |
| Food listing creation & management (Donor) | ✅ Implemented | `app/controllers/food_listing_controller.py` — full CRUD. Donors can create, update, delete own listings. |
| Food request system (Recipient) | ✅ Implemented | `app/controllers/request_controller.py` — recipients request food, donors approve/reject. Business logic in `request_service.py`. |
| Notification system | ✅ Implemented | `app/controllers/notification_controller.py` — notifications created on request status changes. |
| Admin user management | ✅ Implemented | Admin-only route `require_admin` guards `/api/admin/*`. Admins can list and manage users. |
| Role-based dashboards | ✅ Implemented | Frontend: `AdminPage`, `MyListingsPage`, `RequestsPage` — each gated by role from `AuthContext`. |
| JWT authentication with refresh tokens | ✅ Implemented | `app/core/jwt.py` — signed JWT access token (30 min) + opaque refresh token in httpOnly cookie (7 days). |
| Listing search/browse | ✅ Implemented | `HomePage` with listing cards, filtering by category/location. `GET /api/listings` with query params. |
| Profile management | ✅ Implemented | `ProfilePage` + `PUT /api/users/me` endpoint. |

## Security & Authentication
- Password hashing: ✅ bcrypt via `passlib[bcrypt]` (`bcrypt==4.0.1` in requirements)
- Token type: ✅ Real JWT signed with HMAC-SHA256 via `python-jose[cryptography]`, 30-min expiry
- Refresh token: ✅ Cryptographically secure opaque token (`secrets.token_urlsafe(32)`) stored in DB, delivered via httpOnly cookie
- Protected routes: ✅ `get_current_user_id`, `get_current_claims`, `require_admin`, `require_role(*roles)` dependencies used throughout
- Role enforcement: ✅ `require_admin` raises HTTP 403 for non-admins. `require_role("DONOR")` guards listing creation. Recipients blocked from donor-only actions.
- Backend protection: ✅ All write routes protected server-side — not just frontend redirect

## Data Persistence
- Storage method: ✅ JSON file system — `backend/data/` contains `users.json`, `listings.json`, `requests.json`, `notifications.json`, `refresh_tokens.json`. All read and written at runtime via `JsonRepository` base class.
- Frontend-backend integration: ✅ Fully wired — `frontend/src/services/api.ts` uses axios with `VITE_API_BASE_URL`. All pages call backend endpoints.

## Runnability
- Backend: ✅ Started successfully — `uvicorn main:app --port 8010` → `Application startup complete`
- Frontend: ✅ Started successfully — `npm run dev` → Vite v6.4.3, HTTP 200, 1597ms
- API wiring: ✅ Frontend calls backend via axios. `api.ts` centralizes all API calls with auth interceptors.

## Observations
- **Exceptional architecture**: Full layered pattern — controllers → services → repositories → JSON storage. Every layer has a clear responsibility.
- **JWT implementation**: Dual-token strategy (short-lived signed JWT + revocable opaque refresh token in httpOnly cookie) is production-grade design rarely seen in student projects.
- **Role system**: Three roles (DONOR, RECIPIENT, ADMIN) each with distinct capabilities and enforced server-side with reusable `require_role()` factory.
- **Code documentation**: `jwt.py` contains detailed inline documentation explaining JWT internals, token strategy trade-offs, and role design rationale — shows deep understanding.
- **TypeScript**: Full TypeScript frontend with proper interfaces, hooks (`useListings.ts`, `useRequests.ts`), and context (`AuthContext.tsx`).
- **`.env.example`** present — good practice, `.env` is not committed.
- **venv committed**: The `.venv` directory is in the repo (though in `.gitignore`). This bloats the repository significantly.

## Future Scope
- Remove the `.venv` folder from the repository — it's in `.gitignore` but was still committed. Run `git rm -r --cached backend/.venv` to fix.
- Consider adding input validation to listing forms (min/max character limits, image file type checks).
- The JSON file storage works well for this scale; document this architectural decision clearly for future contributors.

## Additional Code-Review Findings

- **No automated tests anywhere.** There is no `test_*.py` under `backend/` and no `*.test.tsx` / `*.spec.ts` under `frontend/src` — the controllers, services, and repositories have zero executable verification despite the clean layering. Even a handful of service-level tests (e.g., request approval rules in `request_service.py`) would significantly raise confidence.
- **Hardcoded fallback secrets.** `backend/app/core/config.py` defaults `JWT_SECRET_KEY = "dev-only-secret-change-me"` and `PASSWORD_PEPPER = "dev-only-pepper-change-me"`. If `.env` is missing, the API silently signs tokens and peppers passwords with publicly known values. Fail fast on startup when these are unset instead of defaulting.
- **Genuine hardening worth praising:** mixing a server-side `PASSWORD_PEPPER` into every password before bcrypt hashing is defense-in-depth that most student projects never consider.
- **Good API documentation practice:** `postman/FoodBridge.postman_collection.json` is committed — a runnable collection doubles as living API documentation for anyone reviewing or extending the project.
- **Duplicated files that can drift:** `backend/main.py` is a one-line shim re-exporting `app.main:app`, and seed logic exists twice (`seed.py` at the repo root and `backend/scripts/seed.py`). Keep one canonical entry point and one canonical seed script to avoid confusion and divergence.
- `seed.py`'s docstring prints plaintext demo credentials (`Admin123`, `Donor123`, `Recipient123`). Acceptable for local demos, but make sure these accounts and passwords are never reused in any deployed environment.

## Detailed Feedback (Instructor Review)

**What you did well:** This is a genuinely strong individual project. You implemented real authentication the right way — bcrypt password hashing via passlib, signed JWT access tokens with python-jose, and a revocable opaque refresh token delivered through an httpOnly cookie. That dual-token design is closer to production practice than most student work I see. Your layered architecture (controllers → services → repositories → JSON storage) is clean and consistent, and role enforcement happens server-side through reusable `require_admin` / `require_role()` dependencies — not just frontend redirects. The axios-based `api.ts` with auth interceptors shows you understand how a frontend should talk to a secured API. Your 45 commits over 6 active days demonstrate steady, genuine work.

**Where to grow:** Two issues. First, you committed the `.venv` directory despite it being in `.gitignore` — this bloats the repo and shows a gap in your git hygiene; fix it with `git rm -r --cached`. Second, input validation on listing forms is thin: no length limits, no file type checks.

**future scope ideas:** Clean the venv from history, add pydantic-level validation constraints to your schemas, and write a short architecture note justifying JSON file storage. Your foundation is excellent — polish it.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
