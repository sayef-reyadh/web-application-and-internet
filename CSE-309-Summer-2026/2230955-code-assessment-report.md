# Code Assessment Report

**Student:** Biozid Bhuiyan Tonoy
**ID:** 2230955
**Section:** Section 5
**Project:** EasyTrip BD – Tourism & Hotel Booking Platform
**Project Type:** Individual
**GitHub:** https://github.com/Biozidtonoy/easyTrip-BD

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`app/main.py`; 9 routers registered) with layered architecture
- Frontend: ✅ React + TypeScript confirmed (`"react"` + `"typescript"` + Vite in `frontend/package.json`)

## Assessed at Commit
- **SHA:** `640b576` — 2026-08-12 — "update readme file (#91)"
- **Repo:** 48 commits, 18 active days (2026-06-12 → 2026-08-12), single author (Biozid Bhuiyan Tonoy), PR-based workflow (PRs #20–#91), GitHub Issues + Kanban tracking, Docker + CI/CD

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Hotel management | ✅ Implemented | `api/hotels.py` + `services/hotel.py` + `crud/hotel.py`; owner-scoped CRUD, `HotelForm`/`HotelEditForm` pages. |
| Destination management | ✅ Implemented | `api/destinations.py` + service/crud; public browsing + admin CRUD (`AdminDestinationsPage`, `DestinationForm`). |
| Room management + availability | ✅ Implemented | `api/rooms.py` — create/update/delete, availability control, hotel-room relationships; `OwnerRoomsPage`. |
| Room images via Cloudinary | ✅ Implemented | `api/room_images.py` + `utils/file_upload.py` + `core/cloudinary_config.py`; `RoomImageManager.tsx`; DB stores cloud URLs (migration `f97091f5e6f9`). |
| Bookings (full workflow) | ✅ Implemented | `api/bookings.py` (152 lines) + `services/booking.py` (402 lines): date validation, availability conflict detection (409), total price calc, booking reference `BK-YYYYMMDD-XXXXXXXX`, traveler create / owner confirm / reject with reason, update/delete, ownership scoping. |
| Reviews | ✅ Implemented | `api/reviews.py` + service/crud; migration `490df51f5e99`. |
| Hotel Owner Applications | ✅ Implemented | Users apply to become owners; admin reviews/approves/rejects (migration `0429a85c9075` adds rejection reason); `AdminApplicationsPage`, `ApplicationStatusPage`, `BecomePartnerPage`. |
| Auth + Profiles | ✅ Implemented | Register/login (`api/auth.py`), profile management (`api/users.py`, `ProfilePage`). |
| Admin dashboard | ✅ Implemented | `AdminDashboardPage.tsx` + stats. |

## Security & Authentication
- Password hashing: ✅ **Argon2** via `pwdlib` (`PasswordHash.recommended()`) in `core/security.py`
- Token type: ✅ Real JWT (`python-jose`, HS256, exp claim, 60-min expiry, `sub` = email)
- Protected routes: ✅ `get_current_user` dependency (OAuth2PasswordBearer on `/auth/login`) on all sensitive endpoints
- RBAC: ✅ `require_roles(*roles)` dependency factory — 403 on role mismatch ("You do not have permission to perform this action.")
- Ownership checks: ✅ `services/booking.py` — travelers can only access their own bookings (403 otherwise), owner endpoints scoped via `get_owner_booking`; only TRAVELER role can create bookings
- Secrets: ✅ All via `pydantic-settings` env config (`DATABASE_URL`, `SECRET_KEY`, Cloudinary keys, `FRONTEND_URL`); no `.env` or real keys committed
- CORS: ✅ Restricted to `localhost:5173/4173` + `FRONTEND_URL` with `allow_credentials=True`

## Data Persistence
- Storage: ✅ PostgreSQL via SQLAlchemy; **15 Alembic migrations** covering users, roles, destinations, hotels, rooms, room images, bookings, reviews, owner applications (each incremental and named)
- Frontend wiring: ✅ Full service layer (`services/*.ts` → `api/endpoints.ts` + axios instance), typed (`types/*.ts`), role-based route guards (`ProtectedRoute`, `RoleProtectedRoute`); no hardcoded data as persistence substitute (static `data/` files only used for homepage marketing sections)

## Runnability & Deployment
- Backend: ✅ All 58 Python files pass `py_compile`; app factory is deployable to AWS Lambda via `Mangum`; Docker + CI/CD pipeline documented
- Frontend: ✅ TypeScript + Vite with `vercel.json` deploy config; static review only

## Observations
- **Textbook layering**: `api/` → `services/` → `crud/` → `models/` + `schemas/` + `enums/` — the cleanest service-architecture split seen so far in this class.
- **Booking logic is genuinely sophisticated**: overlap detection that skips cancelled bookings, server-side price computation with `Decimal`, unique booking references.
- **Professional process**: PRs tied to issue numbers, Kanban tracking, Docker reproducibility (used to solve Lambda packaging inconsistency), CI/CD automation — all described honestly with real debugging anecdotes (Alembic migration errors, JWT issues, Cloudinary ops).
- Docs: 20 numbered documentation files (PRD, SRS, ERD, DFD, TDD, API design, etc.).

## Future Scope
- Add refresh-token rotation or at least token revocation for long-lived sessions (currently single 60-min access token).
- Consider rate-limiting on `/auth/login` to slow credential-stuffing.
- Hotel image uploads live on Cloudinary but destination images still show local `uploads/` folders — migrate fully to cloud storage for consistency.

## Additional Code-Review Findings

- **Duplicated module body in `backend/app/utils/file_upload.py`**: the imports, `cloudinary.config(...)` call, `ALLOWED_IMAGE_TYPES` set, and the entire `save_image()` function appear twice back-to-back before `delete_image()`. Python silently rebinds the second definition, so the app works, but this dead duplicate should be removed — it is a maintenance hazard for anyone editing the "first" copy.
- **Upload validation trusts the client-supplied MIME type only** (`backend/app/utils/file_upload.py`): `save_image()` checks `image.content_type`, which an attacker controls via the request header. There is no file-size limit, no extension check, and no magic-byte sniffing before the stream is forwarded to Cloudinary. Add a size cap and verify content server-side.
- **`uploads/` is git-ignored but 20 binary images are still committed**: `backend/.gitignore` lists `uploads/`, yet ~15 destination, 3 hotel, and 2 room JPEGs remain tracked under `backend/uploads/` (committed before the ignore rule). They bloat every clone and contradict the Cloudinary migration — remove them from history/tracking with `git rm --cached`.
- **Registration endpoint is a user-enumeration oracle** (`backend/app/api/auth.py`): `POST /auth/register` returns a distinct `400 "Email already registered"` for existing accounts, while login correctly uses a generic "Incorrect email or password". Registration should also avoid confirming whether an email exists (e.g., email-verification flow or generic response). Note also that login skips the Argon2 verify for unknown emails, leaking account existence via response timing.
- **Foreign-key columns are unindexed in Alembic migrations**: e.g., migration `1d3b0f12f0c5_create_bookings_table.py` creates only the unique `ix_bookings_booking_reference` index; `traveler_id` and `room_id` have no indexes, yet `get_bookings_by_traveler()` (used by the review service) filters on `traveler_id` — this will degrade to sequential scans as data grows.
- **Positive: review integrity is enforced server-side**: `backend/app/services/review.py` rejects reviews from non-travelers (403), prevents duplicate reviews per hotel (409), and requires a `COMPLETED` booking at that hotel before a review is accepted — a genuinely thoughtful business rule.

## Detailed Feedback (Instructor Review)

**What you did well**
One of the strongest individual submissions in the class. Nine routers over a textbook `api/` → `services/` → `crud/` → `models/` layering, 15 incremental Alembic migrations against PostgreSQL, Argon2 password hashing, JWT auth, a `require_roles()` factory with 403 enforcement, and ownership-scoped booking access. The booking engine is genuinely sophisticated: date validation, overlap-conflict detection returning 409, server-side `Decimal` price computation, and unique booking references. Cloudinary uploads, env-driven secrets, restricted CORS, Docker, CI/CD, and a PR-per-issue workflow with Kanban tracking complete a professional process.

**Where to grow**
Sessions rely on a single 60-minute access token with no refresh or revocation path, and `/auth/login` has no rate limiting. Destination images still live in local `uploads/` folders while room images moved to Cloudinary — an inconsistency. The most notable absence is automated tests: the layering you built makes services trivially unit-testable.

future scope ideas: add refresh-token rotation (or at least revocation), rate-limit the login endpoint, finish migrating all media to cloud storage, and write a pytest suite for the booking service — it is the main thing missing from an otherwise complete engineering package.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
