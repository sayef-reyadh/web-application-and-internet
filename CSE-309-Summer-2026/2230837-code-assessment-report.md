# Code Assessment Report

**Student:** Md. Mahedi Hasan Mahin
**ID:** 2230837
**Section:** Section 5 (Group S5-27)
**Project:** Blood Donation Management System ("শেষ আশা" / Blood Link)
**Project Type:** Team (with Mansura Rashid Rime, 2230260)
**GitHub:** https://github.com/mansurarashidrime50/Blood-Donation-Management-System

---

## Tech Stack
- Backend: ✅ FastAPI (root `backend/` — endpoints/repositories/models/services layering, async SQLAlchemy)
- Frontend: ✅ React + Vite + Tailwind (root `frontend/` — admin/donor/patient/shared trees)

## Assessed at Commit
- **SHA:** `165fc50` — 2026-08-13 — "Implement donor UPDATE (PUT) API and generate project submission reports"
- **Repo:** 60 commits, 10 active days (2026-06-17 → 2026-08-13)
- **This student:** 35 commits (`mahin1710` 15 + `Md. Mahedi Hasan Mahin` 20) — sole author of all feature code

## Individual Contribution — verified per-path authorship

| Claimed (report PDF) | Git record |
|----------------------|-----------|
| Admin module (dashboard, user mgmt, request status mgmt, notifications) | ✅ Verifiable — `backend/app/api/endpoints/admin.py` (11 routes, role-guarded), `frontend/src/admin/` pages |
| Donor module (registration, auth, profile, availability, matching) | ✅ Verifiable — `Donor/` tree + `backend/app/api/endpoints/donor.py` (9 routes) |
| Donor–patient communication (messaging & calling) | ✅ Verifiable — `frontend/src/shared/` (2 commits), chat websocket (`core/websocket.py`), communication endpoints in `shared.py` |
| Frontend–backend integration & testing | ✅ Verifiable — `Donor/backend/tests/` — **all 10 tests pass** (run in this session) |
| Admin/Donor docs & issue/PR documentation | ✅ Verifiable — `docs/issue_*.md` (6 issues), `docs/pull_request_*.md` (4 PRs) |

He also authored the patient endpoints (`patient.py`, 8 routes), auth (5 routes), shared (18 routes incl. websocket chat) — i.e., **51 of 51 API routes** and every backend layer. Note: this means the Patient Module code that teammate Mansura Rashid Rime claims in her report is committed under his identity.

## Features Found (51 routes)
- **Admin** (11): dashboard, user list, create patient/donor, activate/ban users, blood-request list + status updates, donations + verification, escalation runner
- **Donor** (9): compatible request list, decline, donations CRUD, confirm-meeting, completed flow
- **Patient** (8): request CRUD, propose-meeting, received-confirmation, meeting list
- **Auth** (5): donor/patient register, login, refresh token, /me
- **Shared** (18): profile CRUD + image upload, donor search, divisions, public stats, notifications (list/read), chat conversations + **websocket** chat, call logs

## Security & Authentication (verified)
- ✅ Native `bcrypt` password hashing (`core/security.py`)
- ✅ Real PyJWT with **access + refresh tokens**, HS256, token-type check, env-driven secret
- ✅ `OAuth2PasswordBearer` + `get_current_user` (async, DB re-query) + inactive-account 403 block
- ✅ **`require_roles()` factory** with `get_current_admin` / `get_current_donor` / `get_current_patient` aliases — role-based 403 enforcement (92 `Depends(` across endpoints)

## Data Persistence
- ✅ Async SQLAlchemy (SQLite local, PostgreSQL/asyncpg in production per report), 12 models (user, profile, blood_request, donation, meeting, chat, communication, notification, declined_request, analytics, activity_log, base), repositories layer, `seed.py`, Docker + docker-compose, profile-image uploads

## Runnability (tested in this session)
- ✅ All 63 root `backend/` Python files pass `py_compile`
- ✅ **all 10 donor-module tests pass** (auth, profile, search) with `pytest-asyncio` per `Donor/backend/pytest.ini`
- Frontend: static review only (deployment config present: Dockerfile, nginx.conf, Vercel)

## Observations
- **The strongest team contribution assessed so far**: near-complete application (51 routes, websocket chat, matching/escalation/notification services) implemented end-to-end by one student, with a real test suite that passes.
- Issue-driven workflow documented: 6 issue docs + 4 PR docs (donor history crash, profile sync, eligibility countdown, distance-radius filter).
- Minor hygiene: committed `backend/blood_donation.db`, duplicated trees in repo root (`Admin/`, `Donor/`, `Patient/`, `feature/`), `dummy61.txt`/`dummy62.txt`, Pydantic v2 deprecation warning in schemas.

## Future Scope
- Clean the duplicated module trees and committed DB/dummy files from the repo root.
- Move the test suite from the `Donor/` sub-tree into the root `backend/tests/` so it runs in CI.

## Additional Code-Review Findings

- **Your own `docker-compose.yml` bypasses your own config discipline**: it hardcodes `SECRET_KEY=production-secret-key-change-it-12345!` and `postgres`/`postgres` credentials in plaintext — while `core/config.py` carefully reads secrets from the environment. Anyone deploying with your compose file ships a publicly known JWT signing key; the file should reference env vars instead of embedding values.
- **Two generations of the backend ship side by side**: the flat legacy modules `backend/app/auth.py`, `app/models.py`, `app/schemas.py`, `app/database.py`, and the entire `app/routers/` package (admin/auth/donor/patient) are dead code — `main.py` wires only `api/endpoints/*`. Deleting the superseded stack would remove a whole layer of confusion about which code is live.
- **Test coverage is narrower than the headline suggests**: the 10 passing tests in `Donor/backend/tests/` cover only donor auth, profile, and search. Zero tests exist for the admin module (11 routes), the patient module (8 routes), the shared/chat/websocket surface (18 routes), or the matching/escalation/notification services — roughly 42 of 51 routes are unverified.
- **There is no CI to run them anyway**: the repository has no `.github/workflows` (or equivalent), so the test suite only ever runs when someone remembers to run it by hand. Wiring pytest into a push-triggered workflow is cheap and would have caught regressions on every PR you merged.
- **The websocket chat leaks tokens through the URL**: `/chat/ws/{conversation_id}` takes the JWT as a `token` query parameter — bearer tokens in URLs end up in server access logs, proxy logs, and browser history. The participant check you added is genuinely good; move the credential to a subprotocol or header instead.
- **Import hygiene smells in `shared.py`**: mid-file imports (`from app.schemas.communication import ...` partway down the module) and in-function imports (`from app.models.blood_request import BloodRequest` inside `log_call`) suggest circular-import workarounds — the layered repositories pattern you built elsewhere is the right cure.

## Detailed Feedback (Instructor Review)

**What you did well**
You effectively carried this project. All 51 API routes, the full layered backend (endpoints, repositories, models, services), PyJWT access/refresh auth with native bcrypt in `core/security.py`, the `require_roles()` factory with admin/donor/patient aliases and 92 `Depends()` injections, websocket chat, the matching/escalation/notification services, and a pytest suite passing all 10 tests — all committed under your identity. Your issue-and-PR documentation (6 issues, 4 PRs with real bug narratives) shows a professional workflow.

**Where to grow**
The repo root is noisy: duplicated `Admin/`, `Donor/`, `Patient/`, and `feature/` trees, a committed SQLite database, and dummy tracking files. The working test suite lives in the `Donor/` sub-tree instead of the root backend, so it will not run in CI as-is.

**Attribution note**
The git record credits you with essentially all feature code, including the patient module your teammate claims in her report. In future teams, protect yourself and your teammates by insisting everyone commits under their own identity from day one — the current history creates an attribution dispute you did not need. future scope ideas: consolidate the duplicated trees, move the tests into `backend/tests/`, and wire them into CI.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
