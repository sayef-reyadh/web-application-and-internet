# Code Assessment Report

**Student:** Mahmudul Hasan
**ID:** 2020709
**Section:** Section 6
**Project:** Employee Leave & Attendance Management System
**Project Type:** Individual
**GitHub:** https://github.com/Mahmudul-Hasan97/Employee-leave-attendance-management-system

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/`, 5 router modules + crud.py)
- Frontend: ✅ React confirmed (Vite, `frontend/src/`, 8 pages)

## Assessed at Commit
- **SHA:** `36b7f5b` — 2026-08-13 — "Fixed frontend base URL and backend connection"
- **Repo:** 66 commits, **8 active days** (2026-06-27 → 2026-08-13), issue/PR workflow (issues #20–#30)

## Features Found
- Auth: login with role check (employee/admin)
- Leave management: apply, view mine, admin list, approve/reject, delete (5 routes)
- Attendance: clock in/out, status, records (4 routes)
- Employees, dashboard stats, reports
- Frontend: 8 Vite pages (Login, Dashboard, Employees, Leaves, Attendance, Reports, Profile, Settings) + Layout + api.js

## Security & Authentication (verified)
- ❌ **Login returns a hardcoded fake token**: `"token": "fake-jwt-token-string"` (routers/auth.py) — the frontend stores it, but `get_current_user` (dependencies.py) decodes real JWTs with `jose` against `SECRET_KEY`, so **every protected route 401s for real logins — auth is functionally broken**
- ❌ **`require_admin` is never used**: leaves/attendance/employees routes only depend on `get_db` — zero guards
- ✅ bcrypt via passlib used at login verification and seed (admin/admin123, employee/123456 seeded)
- ⚠️ `utils.py` contains vestigial plaintext helpers (`get_password_hash` returns the password unchanged) — not used by auth flow
- ⚠️ `SECRET_KEY` default `"your-super-secret-key-change-this-in-production"` (env-overridable)

## Data Persistence
- ✅ SQLAlchemy + SQLite (`employee_system_v2.db`), User/Attendance/LeaveRequest models, crud layer, seed script, database schema docs + sample SQL

## Runnability (tested in this session)
- ⚠️ `pytest tests` → **1 passed, 1 failed** (test_login_invalid_credentials expects 401 but hits a wrong route and gets 200; test posts to `/auth/login` while the router is mounted at `/api/login`)

## Observations
- Large structured codebase with real CRUD depth and clean layering; CI attempts documented in commit history; but the auth path was left in a broken demo state ("fake-jwt-token-string") and backend routes are unguarded.

## Future Scope
- Issue a real `create_access_token(...)` in login; apply `get_current_user`/`require_admin` to every protected route; fix the test URL and make tests pass; remove plaintext helpers from utils.py.

## Additional Code-Review Findings

- **Admin-created employees get plaintext passwords.** `POST /api/admin/employees` calls `crud.create_user()` in [crud.py](repo/backend/app/crud.py), which does `password=user.password` with no hashing — bcrypt is only applied in the seed script. Every employee added through the admin endpoint is stored in cleartext and, worse, can never log in because login runs `pwd_context.verify()` against that plaintext value.
- **Straightforward IDOR on personal data.** `GET /api/leave/my/{user_id}` ([leaves.py](repo/backend/app/routers/leaves.py)) and `GET /api/attendance/my/{user_id}` ([attendance.py](repo/backend/app/routers/attendance.py)) take the user id from the URL path with no authentication at all — incrementing a sequential integer exposes any employee's leave and attendance history.
- **Identity comes from the request body, not a token.** `LeaveRequestCreate` and `AttendanceCreate` ([schemas.py](repo/backend/app/schemas.py)) both accept `user_id` from the client, so anyone can file a leave request or clock in *as another employee*.
- **Clock-out is unreachable through the API.** `AttendanceUpdate` defines `clock_out`, but `crud.update_attendance(db, attendance_id, data.status)` only ever writes `status`, and `create_attendance` hardcodes `status="Present"` plus `date=str(date.today())` with no duplicate-clock-in guard — the advertised clock in/out workflow cannot actually record a clock-out.
- **The live database is committed.** `backend/employee_system_v2.db` — the real SQLite file containing the seeded accounts — is in the repository instead of being created by `seed.py` at setup time.

## Detailed Feedback (Instructor Review)

**What you did well.** This is a substantial, well-layered codebase: 66 commits across 8 active days with an issue/PR workflow, five feature areas (leave, attendance, employees, dashboard, reports), a clean router/CRUD/model separation, bcrypt used correctly at login, and seed data plus schema documentation. The bones of a real system are here.

**Where to grow.** Your authentication is functionally broken, and that is a serious failure. The login endpoint literally returns the string `"fake-jwt-token-string"`, while a fully correct jose-based `get_current_user` and `require_admin` sit unused in `dependencies.py` — so every protected route would reject a real login with a 401, and all 17 endpoints are actually unguarded. You built the lock and then left the door open. Worse, `utils.py` still contains a plaintext "hashing" helper, and your own test suite fails (1 of 2 tests) because the test posts to the wrong route — evidence you were not running your tests.

**future scope ideas:** make login issue a real token with `create_access_token`, apply `get_current_user`/`require_admin` to every route, delete the plaintext helpers, fix the test URL, and do not consider any feature "done" until its tests pass.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
