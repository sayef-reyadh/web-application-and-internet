# Code Assessment Report

**Student:** Momotaj Akther Happy
**ID:** 2430798
**Section:** Section 6
**Project:** Departmental Student on Duty (SoD) Management System
**Project Type:** Team (with Mohammad Zaid Iqbal Fahad, 2430825)
**GitHub:** https://github.com/Momotaj-Happy/CSE-309A-WebProject-SoD-Management-System

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`back-end/main.py`, 8 routers: auth, users, tasks, schedule, swaps, billing, export)
- Frontend: ✅ React + TypeScript confirmed (Vite, 18 components + pages)

## Assessed at Commit
- **SHA:** `d16edc0` — 2026-08-12 — "Merge pull request #51 release-v1.0-main-deployment"
- **Repo:** 45 commits, 6 active days (2026-06-17 → 2026-08-12)
- **This student:** 25 commits (10 authorships as `Momotaj Akther` + 8 as `Momotaj-Happy` + merge commits), 6 distinct days
- **Teammate (Zaid Fahad):** 20 commits, 5 days

## Individual Contribution — verified per-path authorship

| Claimed (report `docs/25-project-submission-report-happy.md`) | Git record |
|----------------------|-----------|
| IRAS schedule parser engine (`ParserService`) | ✅ `services/parser_service.py` 5 commits (4× Momotaj Akther + 1× Momotaj-Happy) |
| Academic schedule CRUD + unavailable slots | ✅ `services/schedule_service.py` 2/2, `routes/schedule.py` 3 commits (2+1) |
| Shift swap engine + immutable audit log | ✅ `services/swap_service.py` 1/1, `routes/swaps.py` 1 of 2, `ShiftSwapFeed.tsx` 1/1, `test_shift_swap_engine.py` 1 of 2 |
| Student monthly billing & submission | ✅ `services/billing_service.py` 2 of 3, `routes/billing.py` 2 of 3 |
| Frontend setup + UI architecture | ✅ `front-end/` scaffolding (docs/23-work-distribution.md 3/3 commits, sole author) |
| Auth dependency fix, sanitization bug fixes, docs | ✅ `main.py` 5 of 9 commits, docs lead |

Her claimed modules are backed by the commit history — genuine, substantial, full-stack contribution to the team project.

## Features Found (her scope wired end-to-end)
- IRAS schedule text parser (multi-line + tab + regex strategies, day-code normalization)
- Schedule persistence/CRUD (`_SCHEDULES_DB`), 7-day weekly grid UI, unavailable-slot modal
- Shift swap feed with atomic `OPEN → ACCEPTED/CANCELLED` transitions + audit trail UI
- Monthly billing pipeline (DRAFT → SUBMITTED → VERIFIED → APPROVED/REJECTED) with bill summary UI
- Full team app also includes: tasks CRUD, users admin + role assignment, CSV export, RBACMatrix UI

## Security & Authentication (verified)
- ✅ Real JWT (PyJWT, HS256, exp + iat), HTTPBearer
- ✅ PBKDF2-SHA256 (100,000 iterations) with per-user random salt — strong hashing
- ⚠️ Role checks exist inline on some routes (role update → managers only 403; delete user → DEPT_MGR; `/schedule/student/{id}` → Faculty/Managers) but **enforcement is incomplete**:
  - `PATCH /bills/{id}/verify|approve|reject` — takes `Depends(get_current_user)` but **never checks role**; any authenticated user can approve bills
  - Task create/delete/status — any authenticated user can create/delete/assign duty tasks
  - `GET /users`, `GET /tasks/swaps/audit-log` — token-only, no role restriction
- ⚠️ `SECRET_KEY` hardcoded fallback in `auth_service.py`

## Data Persistence
- ⚠️ **All storage is in-memory dicts** (`_USERS_DB`, `_TASKS_DB`, `_SCHEDULES_DB`, `_SWAPS_DB`, `_AUDIT_LOG_DB`, `_BILLS_DB`) seeded with demo users/tasks/schedules/bills. **No database, no file storage** — all data is lost on server restart. The report's claim of "SQLAlchemy ORM persistence" is **not present in the code** (no sqlalchemy import anywhere).
- Frontend wiring: ✅ real API calls through typed services; no hardcoded result arrays.

## Runnability (tested in this session)
- ✅ **26/26 tests pass** (`tests/` — auth, billing pipeline, schedule engine, swap engine, export audit trail, task conflicts)
- ✅ Backend imports cleanly; no compile errors

## Observations
- Very strong scope and process for a team project: 24 docs, issue-driven branches (`username/#issue-feature`), 6 test suites, Pydantic models, layered services.
- The in-memory store is the single biggest weakness — a demo-data app that cannot survive a restart; make it a real DB (SQLAlchemy/SQLite) to reach persistence 2.

## Future Scope
- Replace in-memory dicts with SQLite/SQLAlchemy (or JSON file persistence) — this is the main lost point.
- Add role checks to bill verify/approve/reject and task mutation routes; move `SECRET_KEY` out of code into `.env` only.

## Additional Code-Review Findings

- The billing state machine is not enforced: `verify_bill`, `approve_bill`, and `reject_bill` in `back-end/api/services/billing_service.py` set the new status **unconditionally**, with no check of the current status. A DRAFT bill can be APPROVED directly (skipping submission and verification entirely), and a REJECTED bill can be re-approved — the DRAFT → SUBMITTED → VERIFIED → APPROVED/REJECTED pipeline exists only in the route docstrings.
- Bill totals are fabricated, not calculated: `get_or_create_current_bill` hardcodes `hrs = 3.0` for every completed task (`back-end/api/services/billing_service.py`), so `total_hours` and `total_amount` are 3.0 × rate × task count regardless of the tasks' actual scheduled durations.
- `get_current_user` in `back-end/api/services/auth_service.py` silently **fabricates a user from unverified token claims** when the store lookup fails (`UserService.get_by_id` returns `None`), defaulting to `"System User"` / role `"STUDENT"`. A token belonging to a deleted or unknown user keeps working, and its role comes from the (possibly stale) token rather than the current record — the opposite of the DB-re-query pattern used by stronger submissions.
- `GET /bills/student/{student_id}` (`back-end/api/routes/billing.py`) has no ownership or role check — any authenticated user can read **any** student's full bill history (an IDOR, separate from the missing role checks on verify/approve/reject already noted).
- Mock fallbacks are baked into production routes: `back-end/api/routes/billing.py` uses `payload.get("sub", "mock-1")` and `payload.get("full_name", "Momotaj Happy")`, so a token missing those claims silently reads or creates bills for the hardcoded demo student.
- All four seeded demo users in `back-end/api/services/user_service.py` share the same password `Password123!`, and tokens live for 24 hours (`ACCESS_TOKEN_EXPIRE_MINUTES = 60 * 24`) with no revocation — combined with the in-memory store, every restart resurrects known-credential accounts.

## Detailed Feedback (Instructor Review)

**What you did well.** Your contribution is verified by the git record: the IRAS schedule parser, schedule CRUD with the weekly grid, the shift-swap engine with its audit trail, and the monthly billing pipeline are genuinely yours, and they are wired end-to-end to a real React frontend. Twenty-six passing tests and issue-driven branching show strong engineering process, and the JWT plus PBKDF2 hashing are real.

**Where to grow.** Two serious problems. First, all storage is in-memory dictionaries — nothing survives a server restart, and your report's claim of SQLAlchemy persistence is not present anywhere in the code. Do not claim what you have not built. Second, authorization is incomplete: bill verify/approve/reject and task mutation routes only check that a user is logged in, never their role — any authenticated user can approve bills. The hardcoded SECRET_KEY fallback must also go.

**Attribution note.** Team project with Mohammad Zaid Iqbal Fahad. Per-file commit authorship confirms your claimed modules (parser, schedule, swaps, billing) are your own work; the auth/RBAC infrastructure and conflict engine are his.

**future scope ideas:** replace the in-memory stores with SQLite/SQLAlchemy, then enforce role checks on every bill and task route.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
