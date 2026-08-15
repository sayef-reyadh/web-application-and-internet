# Code Assessment Report

**Student:** Nazmul Alam
**ID:** 2420822
**Section:** Section 6
**Project:** Student Assignment Tracker
**Project Type:** Individual
**GitHub:** https://github.com/tuhinXtg/Student_Assignment_Tracker

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/`, 3 routers + core + database + models + schemas)
- Frontend: ✅ React + TypeScript confirmed (`frontend/`, Vite, 10 pages)

## Assessed at Commit
- **SHA:** `cafd321` — 2026-08-13 — "ci: add JWT environment variables"
- **Repo:** 95 commits, **22 active days** (2026-06-12 → 2026-08-13), single author (Namul Alam Tuhin)

## Features Found (16 routes)
- Auth: register (forced `role="student"`), login
- Courses: CRUD (guarded)
- Assignments: create (self or assigned to a student by teachers), list (role-filtered), detail, update, status patch, delete — with **teacher vs student ownership enforcement on every route** (teacher may only touch own-created assignments; student only their assigned ones, checked server-side)
- Frontend: Student Dashboard (welcome, statistics, progress cards, quick actions), TeacherDashboard, Courses + Edit/AddCourse, Assignment + Add/EditAssignment, Home/Login/Register — wired through typed services (`api.ts`, `auth.ts`, `assignments.ts`, `courses.ts`)

## Security & Authentication (verified)
- ✅ passlib **bcrypt** password hashing
- ✅ Real JWT (python-jose, HS256, exp) with **env-only `SECRET_KEY`** (no hardcoded fallback)
- ✅ `HTTPBearer` + async `get_current_user` that re-queries the DB (Beanie ODM) for the fresh user record
- ✅ Role-based access: registration locked to student; teachers created via `create_teacher.py` admin script; assignment/course routes branch on role + ownership (403-level logic)
- ✅ `.env.example` committed; CI supplies env vars

## Data Persistence
- ✅ **MongoDB** via async **motor + Beanie ODM** (`DATABASE_URL`/`DATABASE_NAME` from env), documents: User, Course, Assignment

## Runnability
- ✅ 17/17 backend Python files pass `py_compile`
- ✅ **GitHub Actions CI** (`.github/workflows/ci.yml`): frontend `npm ci && npm run build` + backend import check on every push/PR to main

## Observations
- One of the most disciplined solo submissions: 95 commits across 22 days (06-12 → 08-13), complete 24-file docs set (PRD, SRS, TDD, ERD/DFD/diagrams), CI pipeline, typed frontend services.
- Clean layering: `core/` (jwt, security, dependencies), `database/`, `models/`, `routes/`, `schemas/`.
- Minor: no backend unit tests (CI only checks import); CI yaml embeds a SECRET_KEY literal (fine for CI, but a rotation note would help).

## Future Scope
- Add pytest coverage for assignment ownership rules; move the CI secret to GitHub Secrets.

## Additional Code-Review Findings

- **Authorization inconsistency bug:** in `backend/app/routes/assignment.py`, `GET /assignments/{assignment_id}` only lets a student view an assignment when `assignment.assigned_to == current_user.id` — but when a student creates their own assignment, `assigned_to` is set to `None`. The list endpoint returns self-created assignments (its `$or` covers `user_id`), so students see their own assignments in the list yet get a **403** when opening them. The detail check should also accept `assignment.user_id == current_user.id`.
- **No `course_id` ownership validation:** `create_assignment` stores whatever `course_id` the client sends without checking that the course exists or belongs to the caller — yet courses are strictly user-scoped everywhere else (`backend/app/routes/courses.py` 403s cross-user access). A user can attach an assignment to another user's course id, creating cross-account data associations.
- **Orphaned assignments on course delete:** `delete_course` removes the course but leaves every assignment referencing that `course_id` dangling — no cascade, no guard, no reassignment.
- `status` and `priority` are unvalidated free strings in `backend/app/schemas/assignment.py` (`status: str = "Pending"`), so typos like `pending` vs `Pending` fragment the dashboard statistics; an enum or `Literal` would make the status workflow reliable. Relatedly, `PATCH /{id}/status` lets a student set *any* string as the status.
- Debug leftovers in `backend/app/routes/auth.py`: `register_user` ends with `print("✅ User inserted successfully!")` and `print(user)`, which writes the full user document — including the **hashed password** — to server logs on every registration.
- `backend/app/core/jwt.py` reads `SECRET_KEY = os.getenv("SECRET_KEY")` with no fail-fast validation: if the variable is missing, the app boots fine and only crashes with a runtime error on the first login attempt. Also note `login_user` returns HTTP **400** (not 401) for invalid credentials, and there is no rate limiting on `/auth/login`.

## Detailed Feedback (Instructor Review)

**What you did well.** An excellent, disciplined submission: 95 commits over 22 active days, a working CI pipeline, and a complete documentation set. The security implementation is textbook — bcrypt via passlib, jose JWTs with an env-only secret key, an async `get_current_user` that re-fetches the user from the database, and server-side role-plus-ownership enforcement on every assignment route: teachers can only touch assignments they created, students only those assigned to them. Locking registration to the student role prevents privilege escalation by design.

**Where to grow.** You have no backend unit tests; CI only verifies that imports succeed, so your ownership rules — the most valuable logic in the project — are untested. Move the CI secret literal into GitHub Secrets and note a rotation policy.

**Attribution note.** Individual project; the commit history supports sole authorship.

**future scope ideas:** write pytest cases covering every 403 branch in the assignment router, secure the CI secret, and keep this layering discipline — `core/`, `database/`, `models/`, `routes/`, `schemas/` is what professional codebases look like.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
