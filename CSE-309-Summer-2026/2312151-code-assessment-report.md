# Code Assessment Report

**Student:** Galib Hasan
**ID:** 2312151
**Section:** Section 6
**Project:** SeatFlow — Event Seat Booking
**Project Type:** Team per student_map (with Ashraful Rifat 2110113 — **0 commits in this repo**; all 50 commits are Galib's)
**GitHub:** https://github.com/galibhasan720/Seat-Flow

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/`, 9 domain modules)
- Frontend: ✅ React + TypeScript confirmed (`frontend/`, Vite + shadcn/ui + Tailwind)

## Assessed at Commit
- **SHA:** `11f6cb9` — 2026-08-08 — "redesign the ui ux (#63)"
- **Repo:** 50 commits, 16 active days (2026-06-12 → 2026-08-08), single author (Galib Hasan 45 + galibhasan720 5)

## Features Found (24 routes, 9 modules)
- Auth (register/login/me, JWT) • Users (profile) • Events (6 routes: discovery, catalog, search, organizer CRUD, booking windows) • Seats (per-event listing, capacity/categories) • Bookings (3 routes: create with seat hold/concurrency, lifecycle, cancel) • Venues (7 routes: CRUD + capacities)
- Frontend: EventsView, EventDetailView, SeatSelectionView, BookingFlowViews, OrganizerView, VenueViews, DashboardView — shadcn/ui design system
- ⚠️ **Stub modules**: `admin`, `analytics`, `notifications` each expose only `/ping` ("proving Router → Controller → Service → Repository") — the roadmap features 12–14 (documented in `project_issues/`) are not yet implemented

## Security & Authentication (verified)
- ✅ bcrypt + real JWT (PyJWT, HS256, `iat` + `exp`, 7-day default)
- ✅ `get_current_user` (Bearer header, DB lookup, `is_active` check → 401/403) + `require_organizer` role guard
- ✅ Events/bookings/venues routes guarded (15–16 guard refs per router); seats listing public by design
- ✅ Env-driven secrets (`JWT_SECRET` → `SUPABASE_JWT_SECRET` → `dev-local-secret-change-me` dev fallback); app **refuses to start** with placeholder DB URL (`is_placeholder_database_url`)

## Data Persistence
- ✅ **Supabase PostgreSQL** (SQLAlchemy + psycopg3), SQL schema files (3 migrations), seed script, docker-compose + Dockerfile, AWS CloudFormation/Terraform/K8s manifests

## Runnability
- ✅ Backend compiles; 2/2 in-memory tests pass (`tests/unit/test_services.py`); full suite (integration/booking flow, event management) requires a live Supabase connection
- Test scaffold: unit + integration suites, fixtures for users/events/bookings

## Observations
- Exceptionally engineered repo: per-module `router → controller → service → repository` layering, custom middleware (CORS/error handler/logging), exception hierarchy, 20 issue-roadmap docs, full infra-as-code (Terraform, CloudFormation, Kubernetes), CONTRIBUTING.md, CI-style scripts (`test-all.sh`).
- Honest scope: admin/analytics/notifications are architecture stubs, not implemented features.
- ⚠️ Teammate Ashraful Rifat (2110113) has no commits in this repository — the team work is not visible here.

## Future Scope
- Implement the admin/analytics/notifications modules (or drop the stubs); make the test suite runnable offline (SQLite fallback) so CI can execute it; coordinate teammate contribution into the repo.

## Additional Code-Review Findings

- **The test suite is largely an empty scaffold.** 12 of the 13 test files under `backend/tests/` are 0 bytes — `test_auth.py`, `test_bookings.py`, `test_events.py`, `unit/test_services.py`, and both `integration/` files contain nothing at all. Only `backend/tests/test_health.py` has content (two smoke tests against `/health` and `/health/db`). The concurrency-critical booking path therefore has zero executable test coverage.
- **Double-booking protection is real but rests entirely on the database.** `backend/app/bookings/service.py` flips `seat.status` to "Booked" after a plain availability read with no row lock (`SELECT ... FOR UPDATE`); correctness comes solely from `UniqueConstraint("seat_id", name="uq_booking_seats_seat_id")` on `BookingSeat` plus the `IntegrityError` → `ConflictError` mapping. Sound on PostgreSQL, but the "seat is not available" error path assumes a race-free read.
- **`cancel()` is carefully designed**: ownership-scoped via `get_for_user` (other users' bookings return 404, leaking no existence information), idempotent on repeat cancellation, and it restores seats to "Available" while deleting join rows to free the unique constraint for rebooking.
- **Weak password policy with no brute-force mitigation.** Registration requires only `min_length=6` (`backend/app/auth/schemas.py`) with no complexity rule, and there is no rate limiter or lockout on the login endpoint (no such dependency in `backend/requirements.txt`).
- **Minor information exposure:** the public `GET /health` endpoint returns `settings.app_env` and `/health/db` reveals database connectivity state (`backend/app/main.py`) to unauthenticated callers; consider gating or trimming these in production.

## Detailed Feedback (Instructor Review)

**What you did well.** This is the strongest-engineered submission in your section. Real bcrypt password hashing and PyJWT tokens with expiry, enforced through `get_current_user` and `require_organizer` guards across the events, bookings, and venues routers. The env-driven secret chain and the refusal to boot against a placeholder database URL show you understand secure configuration, not just syntax. Clean router → controller → service → repository layering, a React 18 frontend, working tests, and infrastructure-as-code put this well beyond a typical course project.

**Where to grow.** Three routers (admin, analytics, notifications) are `/ping` stubs — either implement them or remove them; dead scaffolding in a submission reads as overclaiming. Most of your test suite silently requires a live Supabase connection; add an SQLite fallback so tests run offline. The booking seat-hold concurrency logic deserves a stress test, not just happy-path unit tests.

**Attribution note.** Your teammate has zero commits in this repository and did not submit. Everything here is your work and has been assessed as such — but a two-person scope carried by one person should have been renegotiated earlier.

**future scope ideas:** implement or delete the stub modules, make the test suite offline-runnable, and document the solo effort explicitly.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
