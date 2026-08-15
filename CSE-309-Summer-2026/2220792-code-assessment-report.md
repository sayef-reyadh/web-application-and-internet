# Code Assessment Report

**Student:** Zahid Kabir Utsho
**ID:** 2220792
**Section:** Section 5
**Project:** Student Management System — Attendance & Grade Tracker
**Project Type:** Individual
**GitHub:** https://github.com/Kabir792/student-management-system

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app.py`, 5 routers: auth, student, attendance, grade, report)
- Frontend: ✅ React confirmed (`"react": "^19.2.8"` in `package.json`, JSX components)

## Assessed at Commit
- **SHA:** `07ae8cc2b1a1dee84285f210c8e2c5489a5e3209`
- **Date:** 2026-08-12
- **Message:** "Update README.md with live production Vercel and Render URLs"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 1 (shallow clone) |
| First Commit | 2026-08-12 |
| Last Commit | 2026-08-12 |
| Active Days | 1 |
| Contributors | 1 (Kabir792) |

## Features Claimed vs Found

| Feature | Status | Notes |
|---------|--------|-------|
| User login (admin/teacher roles) | ✅ Implemented | `POST /api/auth/login` — reads users from `data/users.json`. Supports admin and teacher roles. |
| Student management | ✅ Implemented | `student_routes.py` + `student_service.py` — CRUD stored in `data/students.json` with thread-safe file locks. Student registration, listing, editing. `StudentCard.jsx`, `StudentForm.jsx`, `StudentEditModal.jsx` in frontend. |
| Attendance tracking | ✅ Implemented | `attendance_routes.py` + `attendance_service.py` — attendance recorded per student per date in `attendance/attendance.json`. `AttendanceCard.jsx`, `AttendanceForm.jsx`, `AttendanceTable.jsx` in frontend. |
| Grade management | ✅ Implemented | `grade_routes.py` + `grade_service.py` — grades stored in `grades/grades.json`. |
| Transcript / reports | ✅ Implemented | `report_routes.py` — generates per-student reports combining students.json, attendance.json, and grades.json. `StudentTranscriptModal.jsx` fetches `GET /reports/student/{id}`. |
| Role-based UI (admin vs teacher) | ✅ Partial | Login returns role in response, and frontend uses it to show/hide UI elements. However, the backend does not enforce roles — all API endpoints are publicly accessible. |

## Security & Authentication
- Password hashing: ❌ Plaintext — passwords stored as strings in `users.json`, compared with `str(user.get("password")) == password_clean`.
- Token type: ❌ No token issued on login. The login endpoint returns a user dict `{user_id, name, role}` with no JWT or session identifier.
- Protected routes: ❌ Zero `Depends()` usage for auth across all 5 routers. Any request to any endpoint succeeds without logging in.
- Role enforcement: ❌ Roles shown in frontend UI only. Backend performs no authorization checks whatsoever.

## Data Persistence
- Storage method: ⚠️ JSON file-based — all data (students, attendance, grades, users) stored in `.json` files on disk. Files persist across restarts. Uses `threading.Lock()` to prevent concurrent write corruption — a thoughtful design choice.
- Frontend-backend integration: ✅ Fully wired — `attendanceApi.js` uses `fetch()` for all API calls. `StudentTranscriptModal.jsx` calls `GET /reports/student/{id}`. Report generation aggregates all three data files.

## Runnability
- Backend: ✅ Started cleanly — HTTP 200, "Student Management System API v1.0.0".
- Frontend: ✅ Started successfully — Vite v5.4.21, HTTP 200, 942ms.
- Deployment: ✅ Deployed to Render (backend) and Vercel (frontend). `render.yaml` and `vercel.json` included.

## Observations
- **Coherent, well-scoped domain**: A student management system covering attendance, grades, and transcripts is a meaningful, realistic application.
- **Thread-safe JSON file handling**: Using `threading.Lock()` to protect file writes shows awareness of concurrency issues — impressive for a student project.
- **Transcript generation is a strong feature**: Aggregating student data, attendance records, and grades into a single report on demand is non-trivial business logic.
- **JSON files ≠ database**: JSON files persist to disk but lack indexing, transactions, and relational integrity. Switching to SQLite would require minimal code changes.
- **Auth flow is structurally present but incomplete**: The login endpoint works, the role is returned, the frontend respects it — but no session or token secures subsequent requests.
- **Deployed successfully** to two cloud platforms — shows real deployment initiative.

## Future Scope
- Return a JWT from the login endpoint (`python-jose` + `passlib[bcrypt]`). Store the token client-side and send it as `Authorization: Bearer <token>` on all subsequent requests.
- Add `Depends(get_current_user)` to student, attendance, and grade write routes to enforce authentication server-side.
- Replace JSON files with SQLite (via SQLAlchemy) — the thread-lock pattern you already use translates naturally to a DB session pattern.
- Add an admin-only `Depends(require_admin)` to student registration and deletion endpoints to enforce the role distinction you already track.

---

## Additional Code-Review Findings

- **Working credentials are committed to the repository.** `backend/data/users.json` is tracked in git and ships real login passwords (`"admin123"`, `"teacher123"`) together with both teammates' full names and student IDs. Because the app is deployed on Render and reads this exact file at runtime, anyone who browses the public GitHub repo can log in to the live deployment as ADMIN. Change both passwords and remove the file from version control immediately.
- **The login service contains hardcoded alias backdoors.** In `backend/services/auth_service.py`, the alias lists hardcode personal Gmail addresses and first names (`'zahid'`, `'shawon'`, `'admin'`, `'administrator'`, etc.) as valid usernames. Typing the word `admin` as the username authenticates as the administrator. Beyond the security hole, this commits both students' personal email addresses to a public repo.
- **Error responses leak internals.** The same file returns `f"Authentication error: {str(e)}"` to the client on failure — raw exception text (file paths, decode errors) should never leave the server. Log it server-side and return a generic 500 message.
- **Two divergent copies of the student table exist.** `backend/data/students.json` (used by `student_service.py`) and `backend/attendance/students.json` (used by `attendance_service.py`) store the same entity with different schemas (`student_id`/`name` vs `studentId`/`studentName`) and already disagree on data — the same student is in "Software Engineering" in one file and "Computer Science & Engineering" in the other. This is exactly the class of inconsistency a single shared data source prevents. Note also that `data/students.json` commits a personal phone number to git.
- **Duplicated project scaffolding.** The repo root has its own `app.py` (a `sys.path` shim importing `backend.app`), `requirements.txt`, `vercel.json`, and `package.json` alongside the real `backend/` and `frontend/` apps — two parallel project roots makes it unclear which files actually run and invite version drift.
- **No automated tests.** There is no test file anywhere in the repo; even one test asserting that login rejects a wrong password would have been valuable given how central `auth_service.py` is.
- **Fair credit:** the consistent `try/except json.JSONDecodeError` handling in the services and the alias-flexible login UX show genuine thought about robustness — the instincts are right, they just need to be applied to security as well.

## Detailed Feedback (Instructor Review)

**Attribution note (team project):** This was graded on your individual contribution only. Per-commit authorship shows your own work is: authentication, the student registration CRUD/directory module, the PDF transcript generator, and the grade-management UI sync. The attendance and grades *backend* modules were committed by your teammate (Shawon Afrin Badhon), so they were not credited to you. Your verifiable scope is 4 distinct features.

**What you did well**
- The thread-safe JSON file handling (`threading.Lock()`) shows real awareness of concurrency — rare at this level, and it counts as genuine data persistence.
- The transcript/report generator is the strongest feature: it aggregates students, attendance, and grades into a single on-demand report — real, non-trivial business logic.
- Deploying to Render + Vercel demonstrates initiative beyond local development.

**Where to grow (the critical gap is security)**
- Your login compares plaintext passwords from `users.json` and returns **no token**. After login, every API route is publicly accessible — there is no `Depends()` guard anywhere. The roles you return are enforced only in the frontend UI, which any user can bypass by calling the API directly. This is the difference between the appearance of auth and actual auth.
- Concretely: return a signed JWT on login (`python-jose`), hash passwords with bcrypt (`passlib`), and add `Depends(get_current_user)` plus a `require_admin` dependency to your write routes. Your frontend already tracks role — wiring the backend to enforce it is a small step that would transform your security score.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
