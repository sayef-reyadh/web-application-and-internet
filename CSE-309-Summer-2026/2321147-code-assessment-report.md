# Code Assessment Report

**Student:** Tasnia Anjum
**ID:** 2321147
**Section:** Section 5 (Group S5-16)
**Project:** BookEase – Online Appointment & Reservation Booking System
**Project Type:** Individual
**GitHub:** https://github.com/tasnee-stack/CSE309-Project

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`main.py`, 29 routes)
- Frontend: ✅ React + TypeScript confirmed (Vite)

## Assessed at Commit
- **SHA:** `34d71e3` — 2026-08-12 — Merge branch 'feature/21-appointment-crud-enhancements'
- **Repo:** 19 commits, 8 active days (2026-06-12 → 2026-08-12), single author

## Features Claimed vs Found (verified by running the app)

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Customer registration & login | ✅ Implemented | `/api/auth/register` (201), `/api/auth/login` (200), `/api/auth/me` — verified live |
| Shop marketplace (multi-shop) | ✅ Implemented | Shop entity; `GET /api/shops`, `GET /api/services` public (5 seeded services verified) |
| Self-serve business registration | ✅ Implemented | `POST /api/auth/register-business` |
| Service management CRUD | ✅ Implemented | `POST/PUT/PATCH/DELETE /api/services/{id}` |
| Business-hours availability rules | ✅ Implemented | `GET/POST/PUT/DELETE /api/availability` (day-of-week + start/end times) |
| Backend time-slot generation | ✅ Implemented | `GET /api/availability/slots`; 15-min slot interval; booking computes `end_time` from service duration (verified: 45-min Haircut → 10:00–10:45) |
| Double-booking prevention | ✅ Implemented | Same-shop conflict check + cross-shop overlap rule for the same customer (per report §1/§3) |
| Booking CRUD + status state machine | ✅ Implemented | pending → confirmed/rejected/cancel_requested/cancelled/completed; POST verified 201 with pending status + notification message |
| Admin dashboard | ✅ Implemented | `/api/admin/stats` (401 without token — protected), `/api/admin/customers` + activate/deactivate |
| Email notifications | ✅ Implemented | smtplib + BackgroundTasks, graceful "email not configured" fallback in dev |

## Security & Authentication (verified live)
- Password hashing: ✅ passlib `pbkdf2_sha256` (matches report's claim)
- Token type: ✅ Real JWT (`python-jose`, HS256, 720-min expiry)
- Route protection: ✅ `HTTPBearer` + 52 `Depends(` and 13 `require_*` role-guard references across 29 routes
- RBAC: ✅ server-enforced customer vs admin roles; admin endpoints reject anonymous (verified 401)
- Multi-tenant isolation: ✅ strict backend-enforced shop scoping — one shop's admin cannot read/modify another shop's data (per report; claims verified by their own boundary tests)
- Secret handling: ✅ best-in-class — `SECRET_KEY` from env, else git-ignored `.dev_secret_key` file generated once via `secrets.token_urlsafe(48)`; **no hardcoded secret in source**
- `create-admin` CLI utility — no hardcoded credentials anywhere

## Data Persistence
- Storage: ✅ SQLAlchemy over SQLite (`bookease.db`), models for users/services/availability/bookings + schema-migration path preserving existing data
- Frontend wiring: ✅ typed API layer (`API_BASE_URL` via env + dev fallback), live slot selection, admin dashboard — no hardcoded arrays

## Runnability (tested in this session)
- Backend: ✅ `main.py` compiles; app boots, seeds demo data, all critical flows verified live (register → login → public browse → authenticated booking 201 with computed end_time → protected admin 401)
- ⚠️ Committed test file (`tests/test_bookings.py`, 2 tests) FAILS on current code — stale: it posts bookings without an auth token, but booking creation now correctly requires customer auth (verified 401). The report claims a 160+ check suite; only this stale 2-test file is committed.
- Frontend: static review only (single `App.tsx`)

## Observations
- **Very strong, security-conscious solo project.** The single-file design is the main criticism: backend is one 1852-line `main.py` and frontend one 2186-line `App.tsx`. Internally well-organized (section banners, docstrings, Pydantic validators, `RequestValidationError` handler), but monolithic.
- Excellent process artifacts: 22 docs (overview → SRS → ERD → API design), `scripts/` setup+start BAT files, feature branches in history, migration path for schema changes.
- Report quality is unusually reflective (real defect stories: authorization gap on inactive-service filter, data-seeding ordering bug — both caught by their test suite).

## Future Scope
- Split backend into routers/services/models modules and frontend into components/hooks.
- Re-run and re-commit the full test suite (or refresh the two committed tests to log in first) so the repo's tests pass on the final code.

## Additional Code-Review Findings

- **`.gitignore` does not actually exclude the database or the dev secret**: `backend/.gitignore` contains only `venv/`, `__pycache__/`, `*.pyc` — neither `bookease.db` nor `.dev_secret_key` is listed, and both files ship with the submission. The generated dev key is a good pattern, but it must be explicitly git-ignored; the SQLite file with real-looking user data should never be tracked.
- **Double-booking prevention is check-then-insert with no database guarantee**: `create_booking` runs `validate_slot()` + `check_customer_conflict()` and then `db.add()` — two concurrent requests can both pass the overlap checks and commit, double-booking the slot. With no unique/exclusion constraint or transaction serialization, the rule holds only under sequential load.
- **Conflict detection loads rows into Python and leaks an N+1**: `check_customer_conflict` calls `query.all()` and compares times in a loop, touching `other.service.shop.name` lazily per row (one extra query per candidate). An indexed SQL overlap predicate (`appointment_time < :end AND end_time > :start`) with eager loading would be correct and fast.
- **Timezone-naive "past booking" check**: `validate_slot` compares `datetime.combine(booking_date, start) <= datetime.now()` using server-local naive time while a `utcnow()` helper exists elsewhere in the same file — inconsistent clock handling that will accept/reject borderline bookings wrongly once the server and customers are in different timezones.
- **Two enumeration/abuse angles remain**: `POST /api/auth/register` returns `409 "An account with this email already exists"` (user-enumeration oracle), and `create_booking` accepts an arbitrary `customer_email`, so the platform's SMTP account can be used to push email to any address a caller supplies.
- **Positive: deliberate anti-enumeration on bookings**: `load_booking_for` returns 404 (not 403) when a booking belongs to another customer or another shop's admin — deliberately hiding whether the record exists. Combined with past-date rejection and business-hours validation that returns actionable 409 messages, the booking validation layer is genuinely thoughtful.

## Detailed Feedback (Instructor Review)

**What you did well.** This is one of the most security-conscious submissions reviewed. PBKDF2 hashing, real `python-jose` JWTs, `HTTPBearer` with role guards on admin routes, and — rarest of all — genuine multi-tenant isolation where one shop's admin cannot touch another shop's data. Secret handling is exemplary: env-based key with a generated, git-ignored dev fallback, no hardcoded secrets anywhere, and even a `create-admin` CLI instead of seed credentials. Live verification confirmed the full flow: register, login, slot computation from service duration, booking with a proper status state machine, and a 401 on unauthenticated admin access.

**Where to grow.** Bluntly, the structure undermines the substance: a single 1852-line `main.py` and a 2186-line `App.tsx` are not maintainable at this size. More concerning, the only committed tests fail against the current code because they were not updated when booking became authenticated — your report describes a much larger suite, so the repository should prove it. Nineteen commits across eight active days also suggests work compressed into bursts.

**Submission note.** The submitted folder is a snapshot without `.git`; the single-author history was verified via the GitHub API.

**future scope ideas:** split the backend and frontend into modules, and commit a test suite that actually passes.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
