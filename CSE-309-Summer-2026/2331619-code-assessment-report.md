# Code Assessment Report

**Student:** Zainab Akter Rim
**ID:** 2331619
**Section:** Section 5
**Project:** Smart Tenant-Landlord Management Platform
**Project Type:** Team (with Abdullah Mohammad Sadman, 2330322)
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
| Contributors | 2 — Zainab Akter Rim (23) + Zainab-reem (3) = **26 commits**; teammate Abdullah Mohammad Sadman 55 commits |

> Commit history retrieved from the GitHub API (submitted folder contains a snapshot without `.git`). Per-file commit analysis confirms Zainab authored the Tenant and Maintenance Staff modules end-to-end (routers + services), with the database models shared/authored by her teammate.

## Features Claimed vs Found (Zainab's Individual Contribution)

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Tenant Profile | ✅ Implemented | `app/routers/tenant.py` + `tenant_service.py` (Zainab-authored); `TenantProfilePage.tsx`, `TenantHome.tsx`. |
| Maintenance Request workflow | ✅ Implemented | `maintenance.py` router (147 lines, Zainab-authored) + `maintenance_service.py` — full lifecycle: tenant submits with Cloudinary photo uploads, tracks status; staff/landlord manage and update requests. |
| Rental Agreements (viewing) | ✅ Implemented | Tenant-side agreement viewing in `TenantAgreementsPage.tsx`; `agreement.py` router shared with teammate (1 commit each). |
| Complaint Management | ✅ Implemented | `complaint.py` router + `complaint_service.py` (Zainab-authored) — submit, list own, delete (tenant), review + respond (landlord), ownership-scoped. |
| Tenant & Maintenance Staff Dashboards | ✅ Implemented | `dashboard.py` (shared) + `TenantHome.tsx`, `StaffDashboardHome.tsx`, `StaffLayout.tsx` with responsive hamburger sidebar. |
| Notifications | ✅ Implemented | `notification.py` router + `notification_service.py` (Zainab-authored); `NotificationsPage.tsx`, `NotificationPreferencesPage.tsx`. |
| Cloudinary integration (tenant/staff uploads) | ✅ Implemented | `components/tenant/ImageUpload.tsx` wired to Cloudinary uploads for maintenance/photos. |

## Security & Authentication (shared platform work)
- Password hashing: ✅ bcrypt via `passlib` (shared `security.py`)
- Token type: ✅ Real JWT — access (30 min) + refresh (7 days) tokens, hashed refresh-token storage
- Protected routes: ✅ Every route in her routers uses `Depends(get_current_user)` or role-guard dependencies (`tenant_user`, `landlord_user`, `staff_user` — each returns **403** on role mismatch)
- Role enforcement: ✅ Per-endpoint role checks with meaningful 403 messages (e.g., "Tenants only", "Maintenance staff only"); ownership scoping in services (`current_user.id`)
- No secrets committed: only `.github/SECRETS.md` with placeholder documentation

## Data Persistence
- Storage method: ✅ PostgreSQL (Supabase) via SQLAlchemy; Alembic migrations (shared schema including complaints table)
- Frontend-backend integration: ✅ Fully wired — `api/complaint.ts`, `api/maintenance.ts`, `api/notification.ts`, `api/tenant.ts`, `api/agreement.ts` all call the backend with Bearer tokens

## Runnability
- Backend: ✅ All Python files compile cleanly (verified `py_compile` on the full `backend/app` tree).
- Frontend: Static review only — TypeScript + Vite project, deployed at `https://smart-tenant-landlord.vercel.app`.
- API wiring: ✅ Each claimed module has a matching typed API service and page.

## Observations
- **Complete vertical ownership of the Tenant & Staff side**: routers, services, frontend pages, and Cloudinary uploads — 5+ distinct features with real workflow logic, confirmed by per-file git authorship.
- **Mature access-control discipline**: role-guard dependency factories with explicit 403s, and careful static-vs-parameterized route ordering (documented in her report).
- **Team collaboration is real**: shared commits on `auth.py`, `dashboard.py`, `agreement.py`; models authored by teammate — a genuine division of labor consistent with both reports.
- Honest reflection on tradeoffs (reused schema, logic in route handlers, no pagination) shows realistic engineering judgment.
- Her report's environment story (Python 3.14 vs 3.11 venv, bcrypt pinning) matches the repo's pinned `bcrypt==4.0.1` fix.

## Future Scope
- Extract the repeated role-guard dependency factories into the shared `core/dependencies.py` to avoid duplication across routers.
- Standardize on a single API versioning convention (`/api/v1/...` vs plain prefixes) for future maintainability.
- Avoid embedding fallback interfaces in page files (noted in her report) — keep shared types in `src/types/`.

## Additional Code-Review Findings

- **Any maintenance staff member can read any tenant's request.** In [maintenance.py](repo/backend/app/routers/maintenance.py), the `GET /requests/{request_id}` staff branch runs `req = maintenance_service._load_request(db, request_id); return req` with **no assignment check** — unlike the tenant branch (ownership check) and landlord branch (`_assert_request_belongs_to_landlord`). One staff account can enumerate every maintenance request in the system, including tenant details.
- **The router reaches into private service helpers.** The same endpoint calls `maintenance_service._load_request` and `maintenance_service._assert_request_belongs_to_landlord` directly, bypassing the service-layer boundary the rest of the codebase respects and duplicating authorization logic in the HTTP layer.
- **Staff account enumeration by email substring.** `GET /api/v1/maintenance/staff/search?email=` filters `User.email.ilike(f"%{email}%")` — any landlord can probe for staff accounts with arbitrary fragments — and it returns results through the `TenantSearchResult` schema, a tenant DTO reused for staff searches.
- **Duplicate staff assignments are possible.** `assign_staff()` in [maintenance_service.py](repo/backend/app/services/maintenance_service.py) never checks whether the same staff member is already assigned to the request, and the `MaintenanceAssignment` model has no unique constraint on `(request_id, staff_id)` — repeated calls silently create duplicate rows.
- **No automated tests exist anywhere in the repo** (no `tests/` directory, no test files), despite `docs/20-tdd.md` describing a test strategy — the role guards and ownership scoping in your modules are unverified by any executable test.

## Detailed Feedback (Instructor Review)

**What you did well.** You owned the entire tenant and maintenance-staff side of the platform end-to-end — routers, services, React pages, and Cloudinary photo uploads. Your access control is genuinely mature: real JWT access + refresh tokens, bcrypt hashing, and role-guard dependencies that return explicit 403s ("Tenants only", "Maintenance staff only") with ownership scoping in the services. The maintenance workflow with photo upload and status tracking is real workflow logic, not CRUD filler, and your honest write-up of tradeoffs (reused schema, no pagination) shows good engineering judgment.

**Where to grow.** Your role-guard factories are copy-pasted across routers — extract them into the shared dependencies module. API prefixes are inconsistent, there is no pagination on list endpoints, and shared TypeScript types are embedded inside page files. These are polish issues, but at your level you should be fixing them without being told.

**Attribution note.** Per-file commit authorship confirms you wrote the tenant, maintenance, complaint, and notification modules yourself; your teammate authored the shared models, and you collaborated on auth, dashboard, and agreements. This is a credible division of labor.

**future scope ideas:** deduplicate the guards, add pagination, centralize shared types.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
