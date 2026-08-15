# Code Assessment Report

**Student:** Abila Khan Keya
**ID:** 2230715
**Section:** Section 5
**Project:** LifeFlow – Emergency Blood Donation Management System
**Project Type:** Individual
**GitHub:** https://github.com/ke961/Blood-Donation-Management-System-Web

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`app/main.py`, modular routers)
- Frontend: ✅ React 19 confirmed (Vite + React Router DOM v7)

## Assessed at Commit
- **SHA:** `d0ae447` — 2026-08-12 — "perf: instant emergency requests rendering on home page with non-blocking background fetch"
- **Repo:** 58 commits, 7 active days (2026-07-20 → 2026-08-12), single author (Abila Khan)
- Deployed: frontend on Vercel, backend on Render

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Patient Care Hub | ✅ Implemented | `routers/patient.py` (88 lines) — post emergency blood requests (blood type, hospital, urgency), track fulfillment status, view compatible active donors. |
| Voluntary Donor Portal | ✅ Implemented | `routers/donor.py` (331 lines, 17 Depends) — real-time availability toggle, matched-requests query, one-click volunteer commitment, donation history. |
| Admin Portal | ✅ Implemented | `routers/admin.py` (1134 lines, 43 Depends) — system analytics stats, user management + account deletion (FK cascades handled), emergency request posting, donation audit logs, request status updates. |
| Public emergency feed | ✅ Implemented | `/public/emergency-requests` — urgency-sorted (Critical/Urgent first) with blood-group filter; frontend does non-blocking background fetch. |
| Auth + registration | ✅ Implemented | `/auth/register`, `/auth/login`, `/auth/me` — registration restricted to donor/patient (admin role rejected with 400). |
| Role-gated frontend routes | ✅ Implemented | `AdminProtectedRoute.jsx`, `DonorProtectedRoute.jsx`, `PatientProtectedRoute.jsx` + Axios Bearer interceptor (`services/api.js`). |
| Seeded demo environment | ✅ Implemented | `utils/create_admin.py` seeds admin + 4 sample donors + 4 sample requests. |

## Security & Authentication
- Password hashing: ✅ bcrypt (`passlib`)
- Token type: ✅ Real JWT (`python-jose`, HS256, 60-min expiry, role claim)
- Protected routes: ✅ `HTTPBearer` + `get_current_user` + role guards `get_current_admin` / `get_current_donor` / `get_current_patient` (403s)
- Registration hardening: ✅ role whitelist prevents self-registering as admin
- ⚠️ Hardening notes: fallback `SECRET_KEY` default in code (`load_dotenv` used, so env should override) and CORS wildcard `*` + credentials — recommend enumerating origins and failing fast without a secret

## Data Persistence
- Storage: ✅ SQLite via SQLAlchemy — User, BloodRequest, Donation models with proper relationships, explicit `ondelete="CASCADE"` + `cascade="all, delete-orphan"`; auto-migration for missing columns on legacy DBs (`ALTER TABLE ... ADD COLUMN`)
- Frontend wiring: ✅ Axios service layer with token interceptor; no hardcoded data arrays
- Real `blood_donation.db` committed with seeded data

## Runnability
- Backend: ✅ All 13 Python files pass `py_compile`
- Frontend: static review only; `vercel.json` SPA rewrite rules present
- Deployment stories are concrete and real (CORS preflight/401, Vercel SPA 404, SQLite FK IntegrityError, Render cold-start 504 — each with a specific fix)

## Observations
- **Real production deployment**: Vercel + Render with documented debugging of genuine cross-origin/auth/infrastructure issues — the report reads like actual ops experience.
- **Solid solo scope**: three distinct role portals + public feed + audit trails — comfortably exceeds course feature expectations.
- Cleanup notes: `app/auth.py` (387 lines, ~190 commented) and `main.py` (148 lines, ~60 commented) carry large blocks of commented-out dead code from earlier iterations — harmless but noisy.
- Docs: PRD-phase folder, SRS, use-cases, system/API/database design, non-functional requirements, plus an admin-feature issues-and-PRs log.

## Future Scope
- Strip the commented-out code blocks from `auth.py` / `main.py`.
- Make `SECRET_KEY` mandatory via env (no code fallback) and restrict CORS origins.
- Consider expiry revocation for admin role changes (role claim is embedded in the token for up to 60 min).

## Additional Code-Review Findings

- **The CI pipeline named "Test Backend API" runs no tests**: [.github/workflows/deploy.yml](repo/.github/workflows/deploy.yml) has a `test-backend` job that only installs `requirements.txt`, and a `test-frontend` job that only runs `npm run build` — yet both gate production deploys to Render and Vercel. The green checkmark certifies that dependencies install, nothing more.
- **The seeder hardcodes production-reachable credentials**: [create_admin.py](repo/backend/app/utils/create_admin.py) creates `admin@gmail.com` with password `admin123` and four demo donors all sharing `donor123`, and the committed `blood_donation.db` already contains these accounts — if the seed runs in the Render environment, the live admin login is a dictionary word.
- **Role guards never touch the database**: `get_current_user` in [auth.py](repo/backend/app/auth.py) returns the decoded JWT payload directly — no `db.query(User)` — so when the admin deletes an account (the FK-cascade deletion you implemented), that user's token keeps working for the remainder of its 60-minute life. The guards verify a *token*, not a *user*.
- **No validation on the domain's most critical field**: `UserRegister.blood_group` is a free-text `Optional[str]` with no enum check, in an app whose entire matching logic depends on blood-group strings — a typo like `"0+"` or `"O positive"` silently corrupts compatibility matching. `password` is likewise a bare `str` with no minimum-length policy.
- **Deprecated clock API for token expiry**: `create_access_token` uses `datetime.utcnow()`, which is deprecated (Python 3.12+) and naive — use `datetime.now(timezone.utc)` so expiry math is timezone-aware.
- **The dead-code problem is wider than noted**: beyond `app/auth.py` and `main.py`, [routers/auth.py](repo/backend/app/routers/auth.py) also carries ~190 commented lines — two complete earlier versions of the register/login endpoints — so the live auth flow starts at line 190 of its own file.

## Detailed Feedback (Instructor Review)

**What you did well**
This is a strong solo build: three distinct role portals plus a public urgency-sorted emergency feed, python-jose JWT with bcrypt, `HTTPBearer` with role guards returning 403, a registration role whitelist that blocks self-promotion to admin, FK-cascade-safe account deletion, and a genuinely deployed stack (Vercel + Render) with documented debugging of real CORS, SPA-rewrite, and cold-start failures.

**Where to grow**
`auth.py` and `main.py` carry large blocks of commented-out dead code from earlier iterations — strip them. Make `SECRET_KEY` fail fast from the environment instead of falling back to a code default, and replace the CORS wildcard-plus-credentials combination with an enumerated origin list. The 60-minute role claim embedded in tokens also means role changes take up to an hour to take effect.

**Submission note**
Your submission arrived as a shallow clone, so the 58 commits over 7 days documented in your report could not be independently replayed; the code and deployed app were verified directly instead. For future submissions, push the full history so your process is auditable. future scope ideas: remove the dead code, harden secrets and CORS, and add a small pytest suite behind the deployed service.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
