# Code Assessment Report

**Student:** Abdullah Mohammad Sadman
**ID:** 2330322
**Section:** Section 5
**Project:** Smart Tenant-Landlord Management Platform
**Project Type:** Team (with Zainab Akter Rim, 2331619)
**GitHub:** https://github.com/A-M-Sadman/smart-tenant-landlord

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`app/main.py`, routers registered via `APIRouter` with prefixes)
- Frontend: ✅ React confirmed (`"react"` in `package.json`, TypeScript `.tsx` pages with hooks)

## Assessed at Commit
- **SHA:** `f670d91`
- **Date:** 2026-08-12
- **Message:** "Fix/15 integration testing bug fixes (#72)"

## Commit History (Team Repo)
| Metric | Value |
|--------|-------|
| Total Commits | 81 |
| First Commit | 2026-06-09 |
| Last Commit | 2026-08-12 |
| Active Days (repo) | 33 |
| Contributors | 2 — Abdullah Mohammad Sadman (44) + A-M-Sadman (11) = **55 commits**; Zainab Akter Rim (23) + Zainab-reem (3) = 26 commits |

> Commit history retrieved from the GitHub API (submitted folder contains a snapshot without `.git`). Per-file commit analysis confirms the division of labor described in the report: Sadman authored the Property/Unit, Tenant Assignment, Rent Payment, Analytics, Dashboard, and Admin code; auth and the database schema were shared.

## Features Claimed vs Found (Sadman's Individual Contribution)

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Property & Unit Management | ✅ Implemented | `app/models/property.py`, `app/models/unit.py`, `app/routers/property.py` — full CRUD with ownership scoping (`current_user.id`), unit sub-resources. Frontend: `PropertiesPage.tsx`, `PropertyForm.tsx`, `PropertyDetail.tsx`, `UnitForm.tsx`. |
| Tenant Assignment | ✅ Implemented | `app/routers/assignment.py` + `AssignTenantModal.tsx` — links tenant profile to a unit. |
| Rental Agreements (creation) | ✅ Implemented | `app/models/agreement.py` / `app/routers/agreement.py` (shared with teammate; creation flow authored by Sadman) + `CreateAgreementModal.tsx`. |
| Rent Tracking | ✅ Implemented | `app/models/payment.py`, `app/routers/payment.py`, `payment_service.py` (all authored by Sadman) — payment recording + landlord rent-status views. |
| Landlord Dashboard & Admin Panel | ✅ Implemented | `app/routers/dashboard.py`, `app/routers/analytics.py` (Sadman-authored) + Recharts dashboards (`AnalyticsPage.tsx`, `DashboardHome.tsx`), admin pages (`AdminDashboardPage.tsx`, `AdminUsersPage.tsx`, `AdminPropertiesPage.tsx`). |
| Analytics module | ✅ Implemented | `analytics_service.py` + `api/analytics.ts`, cross-role analytics. |
| Railway deployment, Alembic, Cloudinary | ✅ Implemented | `Procfile`, 10+ Alembic migrations, Cloudinary image upload for property photos (`ImageUpload.tsx`). |

## Security & Authentication (shared work, verified)
- Password hashing: ✅ bcrypt via `passlib` (`CryptContext(schemes=["bcrypt"])`)
- Token type: ✅ Real JWT — python-jose HS256, access tokens with expiry (30 min), refresh tokens (7 days) with **hashed refresh-token storage** (`hash_token` via SHA-256)
- Protected routes: ✅ `HTTPBearer` + `Depends(get_current_user)` throughout; every property/unit/agreement/payment route uses `require_role(...)`
- Role enforcement: ✅ `require_role("landlord", "admin")` factory returns **403** for wrong roles; ownership checks scope queries to the current user
- No secrets committed: `.github/SECRETS.md` documents the secret list and env-var setup with placeholder values only

## Data Persistence
- Storage method: ✅ PostgreSQL (Supabase, deployed) via SQLAlchemy; Alembic migration chain covering initial schema, refresh-token indexes, rental agreements, tenant assignments, complaints, profile photos
- Frontend-backend integration: ✅ Fully wired — 12 typed API service files (`api/property.ts`, `api/agreement.ts`, `api/payment.ts`, `api/analytics.ts`, etc.) with Bearer-token auth; role-based frontend layouts (admin/landlord/staff/tenant)

## Runnability
- Backend: ✅ All Python files compile cleanly. Standard FastAPI + SQLAlchemy layout with `Procfile` for deployment.
- Frontend: Static review only — TypeScript + Vite project with `vercel.json`; deployed at `https://smart-tenant-landlord.vercel.app`.
- API wiring: ✅ Every Sadman module has a matching frontend API service and page.

## Observations
- **Production-grade engineering for the Admin/Landlord side**: layered `models → services → schemas → routers`, UUID keys with Pydantic serialization validators, route-ordering discipline, Alembic head-merge workflow, and 33 active days of development.
- **Strong security posture**: JWT access + refresh token rotation with hashed refresh tokens, bcrypt, role-based 403s, and ownership-scoped queries — the `require_role` factory pattern is exactly the pattern the rubric rewards.
- **Honest engineering tradeoffs** documented in the report (no pagination, no repository layer, inconsistent frontend call conventions) — a mature, self-aware reflection.
- Shared work (auth, schema, testing) was genuinely collaborative: per-file history shows both teammates' commits on `auth.py`, `security.py`, `dashboard.py`.
- `pycache` files are committed in the snapshot folder (minor hygiene issue in the submission copy).

## Future Scope
- Add pagination to list endpoints (noted by the student as a known tradeoff).
- Split large routers into smaller modules and standardize frontend API call conventions.
- Ensure `__pycache__`/`venv` artifacts are excluded from any zipped submission (`.gitignore` already covers them in git).

## Additional Code-Review Findings

- **Tenants can mark their own rent as paid with no verification.** `POST /api/v1/payments/{payment_id}/pay` calls `tenant_pay()` in [payment_service.py](repo/backend/app/services/payment_service.py), which sets `payment.status = PaymentStatus.paid` after only an ownership check — no payment-gateway callback, receipt, or landlord confirmation step. As written, a tenant can flip any pending payment to `paid` for free.
- **The JWT secret has an insecure default.** [config.py](repo/backend/app/core/config.py) declares `SECRET_KEY: str = "change-me-in-production"`. If the `.env` file is ever missing in a deployment, every access and refresh token is signed with a publicly known key, and nothing at startup refuses to boot in that state.
- **Image uploads are unbounded.** [images.py](repo/backend/app/routers/images.py) does `contents = await file.read()` with no size limit and trusts only the client-supplied `Content-Type` header before forwarding bytes to Cloudinary — a very large or mislabeled upload is accepted unchecked.
- **Role guards are implemented twice.** [payment.py](repo/backend/app/routers/payment.py) defines its own local `landlord_user`/`tenant_user` dependencies instead of reusing the shared `require_role(...)` factory in `app/core/dependencies.py` — the same authorization rule now lives in two places and can silently drift apart.
- **No automated tests exist anywhere in the repo** (no `tests/` directory, no test files), even though `docs/20-tdd.md` documents a test-driven approach. The services, Alembic chain, and ownership checks are entirely unverified by executable tests.
- **A local build cache is committed**: `frontend/.vite/deps/_metadata.json` is a machine-specific Vite artifact that should never be in version control.

## Detailed Feedback (Instructor Review)

**What you did well.** Your side of this platform is close to production-grade. The layered `models → services → schemas → routers` structure across twelve routers is disciplined, and the security work you shared — `python-jose` access tokens, seven-day refresh tokens stored as SHA-256 hashes, bcrypt, and a `require_role()` factory returning proper 403s — is exactly how authorization should be built. Your Property/Unit, Tenant Assignment, Rent Payment, Analytics, Dashboard, and Admin modules all scope queries by `current_user.id`, and the ten-plus Alembic migrations, Railway `Procfile`, and Cloudinary uploads show real deployment maturity. The honest list of tradeoffs in your report (no pagination, no repository layer) is the kind of self-awareness worth keeping.

**Where to grow.** List endpoints will degrade without pagination. Some routers are large enough to split, and frontend API call conventions are inconsistent across your twelve service files. Fifty-five commits over 24 active days is solid, but the submitted snapshot included `__pycache__` artifacts — sloppy hygiene in a formal submission.

**Attribution note.** Per-file history confirms the division with Zainab: your modules are verifiably yours; auth and the schema were genuinely shared, with both of your commits on `auth.py` and `security.py`.

**future scope ideas:** add pagination, split the large routers, standardize the frontend API layer, and keep build artifacts out of submissions.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
