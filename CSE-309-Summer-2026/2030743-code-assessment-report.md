# Code Assessment Report

**Student:** Md. Rifat Khan Sakil
**ID:** 2030743
**Section:** Section 5
**Project:** Khan Agro Cattle Farm Management System
**Project Type:** Individual
**GitHub:** https://github.com/RifatKhanSakil/Khan-Agro

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/main.py` and `backend/app/main.py`)
- Frontend: ✅ React confirmed (`"react": "^19.2.7"` in `package.json`, JSX components)

## Assessed at Commit
- **SHA:** `9f85a761da1250d2cc88c45ec95f6b8e2ae10164`
- **Date:** 2026-08-11
- **Message:** "Resolve merge conflict for main.py and add bookings router"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 98 |
| First Commit | 2026-06-13 |
| Last Commit | 2026-08-11 |
| Active Days | 11 |
| Contributors | 1 (RifatKhanSakil) |

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Livestock management (cattle/goat listing) | ✅ Implemented | `livestock.py` router with full CRUD. `CattleListingPage.jsx` and `GoatListingPage.jsx` in frontend. 4 routes protected with `Depends(get_current_user)`. |
| Animal detail page | ✅ Implemented | `AnimalDetailsPage.jsx` + `GET /api/animals/{id}` backend route. |
| Eid booking system | ✅ Implemented | `eid_booking.py` backend router + `EidBookingPage.jsx` in frontend. |
| Bookings management | ✅ Implemented | `bookings.py` router added (latest commit). |
| User authentication (login/signup) | ✅ Implemented | `/api/auth/signup` and `/api/auth/signin` backend routes. `AuthPage.jsx` in frontend. Token stored in localStorage and sent as Bearer header. |
| Admin role | ✅ Implemented | `require_admin` dependency in `dependencies.py`. Admin email hardcoded as gate for admin registration. |
| Admin messages / inquiries | ✅ Implemented | `AdminMessagesPage.jsx` + `GET /api/inquiries` backend endpoint (auth-protected). |
| Contact form | ✅ Implemented | `ContactPage.jsx` + `POST /api/inquiries` backend route. |
| Gallery | ✅ Implemented | `gallery.py` router present. |
| Farm info / visiting info / FAQ | ✅ Implemented | `visiting_info.py`, `faq.py`, `about_us.py` routers. `FarmInfoSection.jsx` on homepage. |
| Eid sales / Qurbani prep / Premium Qurbani | ✅ Implemented | Dedicated routers for each: `eid_sales.py`, `qurbani_prep.py`, `premium_qurbani.py`. |

## Security & Authentication
- Password hashing: ❌ Passwords stored as **plaintext** in MongoDB — `"password": payload.password` in `auth.py`. No bcrypt or hashing of any kind.
- Token type: ❌ Fake token — issued as `f"token_{email}"` string (not a real JWT). The token is just the user's email prefixed with `"token_"`.
- Protected routes: ✅ `Depends(get_current_user)` used in `livestock.py` (4 routes) and `Depends(require_admin)` present. The dependency correctly blocks invalid tokens with HTTP 401.
- Role enforcement: ⚠️ Partial — `require_admin` raises HTTP 403 for non-admins. However, the token mechanism itself is insecure (not cryptographically signed), so role checks can be trivially bypassed by constructing a fake token.

## Data Persistence
- Storage method: ✅ MongoDB — `pymongo` used throughout. All collections (users, animals, bookings, inquiries, etc.) persisted to MongoDB Atlas/local instance.
- Frontend-backend integration: ✅ Fully wired — `frontend/src/api.js` calls `http://127.0.0.1:8000` with Bearer token authentication for all data operations. No hardcoded data arrays found.

## Runnability
- Backend: ⚠️ Could not start locally — `import main` hangs because MongoDB connection blocks startup (requires a running MongoDB instance or Atlas connection string). Code is valid but DB-dependent startup.
- Frontend: ✅ Started successfully — Vite v8.1.5, HTTP 200, 4167ms
- API wiring: ✅ Frontend fully wired to backend via `api.js` with Bearer token headers sent on all authenticated requests.

## Observations
- **Exceptional documentation**: 17 markdown documentation files including SRS, ERD, DFD, TDD, user stories, interviews, surveys, acceptance criteria, stakeholder analysis — professional-level project planning.
- **98 commits over 11 active days since June 13** — the strongest development history seen so far. Clear evidence of sustained effort over multiple months.
- **11+ backend routers** covering a wide, realistic domain: livestock, bookings, Eid-specific flows, gallery, FAQ, farm info, qurbani operations.
- **Token mechanism is the key weakness**: The fake `token_email` string bypasses the need for a proper secret key but is trivially forgeable — any user could construct `token_admin@email.com` if they knew the admin email.
- `requirements.txt` exists but appears empty — dependencies not pinned. This is a maintenance risk.

## Future Scope
- Replace the fake token with a real JWT: use `python-jose` to sign tokens with a secret key and set an expiry time. This is a small change that completely transforms security.
- Hash passwords with `passlib[bcrypt]` before storing — this is essential for any real application.
- Pin all dependencies in `requirements.txt` so the project is reproducible without guessing package names.
- The project has excellent scope and documentation — the remaining gap is purely in the security layer.

## Additional Code-Review Findings

- **The signup error message hands attackers the admin account.** `app/routers/auth.py` rejects admin registration with `detail=f"Only '{ADMIN_EMAIL}' is allowed to register as Admin."` — so a single 400 response reveals `adminkhan@gmail.com` to anyone. Combined with the forgeable `f"token_{email}"` scheme, that one error message is literally everything needed to mint an admin session (`token_adminkhan@gmail.com`). Never echo privileged identifiers in error responses.
- **`backend/.env` is committed to git.** It currently holds only `mongodb://localhost:27017` with no credentials, so nothing is exposed yet — but a tracked `.env` is exactly how Atlas credentials end up in permanent history. Remove it from tracking and rely on `backend/.gitignore`. Two `__pycache__/*.pyc` bytecode files are also committed and should be cleaned.
- **Two parallel backend applications exist.** `backend/main.py` is a standalone legacy FastAPI app with its own inline `Inquiry` model importing from `backend/database.py`, while `backend/app/main.py` is the newer structured application using `app/core/database.py`. The legacy copy is dead code that can still be started by mistake — delete it to leave one unambiguous entry point.
- **Import-time database work with swallowed errors.** `backend/database.py` connects and pings MongoDB at module import, and on failure only `print()`s — the app continues running with a broken client, so connection problems resurface later as confusing request-time errors. Connect lazily and fail loudly.
- **Zero automated tests** exist anywhere in the repository — with 13 routers, even a small set of auth/livestock tests would protect the core flows.
- **Worth acknowledging:** the `app/` package is properly layered (`routers/`, `schemas/`, `core/`), and inputs are normalized consistently (`email_clean = payload.email.lower().strip()`) — small habits that prevent real bugs.

## Detailed Feedback (Instructor Review)

**What you did well:** This is one of the most substantial projects in the section. Thirteen routers covering livestock, bookings, Eid flows, gallery, FAQ, and qurbani operations — all persisted to MongoDB, all fully wired to the React frontend with Bearer-token headers. Your 98 commits across 11 active days over two months is the strongest development history I've assessed, and the seventeen documentation files (SRS, ERD, DFD, user stories, acceptance criteria) show planning discipline well beyond the norm. You also correctly used `Depends(get_current_user)` and `require_admin`, so you clearly understand the dependency-injection pattern.

**Where to grow:** Which makes the security flaws harder to excuse. Your "token" is literally the string `f"token_{email}"` — anyone who guesses an email address can forge a session, including the admin's, since the admin email is hardcoded. Passwords sit in MongoDB as plaintext. You built the entire guard structure and then handed out a key that copies itself. Your `requirements.txt` is also effectively empty, so the project is not reproducible.

**future scope ideas:** Replace the fake token with signed JWTs (python-jose), hash passwords with bcrypt, and pin your dependencies. These are small changes — do them, and this becomes a genuinely excellent submission.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
