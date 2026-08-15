# Code Assessment Report

**Student:** Shawon Afrin Badhon
**ID:** 2222625
**Section:** Section 5
**Project:** Student Management System — Attendance & Grade Tracker
**Project Type:** Team (with Zahid Kabir Utsho 2220792)
**GitHub:** https://github.com/Kabir792/Web_App_Project

---

## Tech Stack
- Backend: ✅ FastAPI confirmed
- Frontend: ✅ React (JSX) confirmed

## Assessed at Commit
- **SHA:** `07ae8cc2b1a1dee84285f210c8e2c5489a5e3209`
- **Date:** 2026-08-12
- **Message:** "Update README.md with live production Vercel and Render URLs"

## Commit History (shared team repo)
| Metric | Value |
|--------|-------|
| Repo Total Commits | 14 |
| This Student's Commits | 3 (all on 2026-08-08) |
| Active Days (this student) | 1 |
| Contributors | 2 |

## Your Contribution (per git attribution)
- **Attendance module (full-stack):** backend `attendance_routes.py` + `attendance_service.py`, frontend AttendanceDashboard / Manage pages, `attendanceApi.js` wiring.
- **Grades module (backend only):** `grade_routes.py`, `grade_service.py`. The grades frontend was committed by your teammate.

## Features Found
| Feature | Status | Notes |
|---------|--------|-------|
| Attendance tracking | ✅ Implemented | Full-stack, your commits |
| Grades backend | ✅ Implemented | Backend only; frontend is teammate's |
| Auth / student CRUD / transcripts | — | Teammate's commits, not attributed to you |

## Security & Authentication
- Password hashing: ❌ none in your code
- Token type: ❌ no tokens issued
- Protected routes: ❌ no `Depends()` on your routes
- Role enforcement: ❌ not enforced server-side

## Data Persistence
- Storage method: JSON files (attendance.json, grades.json) with thread-safe locking ✅
- Frontend-backend integration: ✅ attendance frontend wired to your API

## Observations
- Solid, cleanly structured attendance module with proper file locking and full frontend wiring.
- Scope is limited to ~2 modules; no authentication or authorization anywhere in your contribution.
- All your commits landed on a single day.

## Future Scope
- Add authentication (JWT + password hashing) and protect your routes server-side with `Depends()`.
- Spread development over multiple days; commit early and often under your own account.
- Take ownership of a complete feature end-to-end (both backend and frontend) — the grades UI was left to your teammate.

## Additional Code-Review Findings

- **Your grades module is not thread-safe**: the attendance service guards file access with `FILE_LOCK`, but [grade_service.py](repo/backend/grades/grade_service.py) does read-modify-write (`_load_grades` → mutate → `_save_grades`) with no lock at all — two concurrent `POST /api/grades` requests can silently drop one record. The locking you wrote for attendance should have been applied here too.
- **Invalid marks are silently converted to a real grade**: `calculate_grade_letter_and_point` catches `ValueError/TypeError` and coerces bad input to `0.0`, so submitting `"marks": "abc"` stores a legitimate-looking `F` record instead of returning a 400 validation error.
- **Inconsistent response contract in the GPA summary**: `get_student_gpa_summary` returns the key `gpa` when a student has no grades but `cgpa` when they do, and `GET /api/grades/{student_id}` answers 200 `success: true` for a student that does not exist — an API consumer cannot distinguish "no record" from "empty transcript".
- **CORS is wide open**: [app.py](repo/backend/app.py) sets `allow_origins=["*"]` together with `allow_credentials=True` — an invalid and unsafe combination that the report's own README deployment (Render/Vercel) ships to production.
- **No request validation layer**: grade routes parse raw `await request.json()` and the service accepts both `student_id` and `studentId` (`data.get(...) or data.get(...)`), indicating the frontend/backend contract was never formalized — Pydantic models exist in the framework you chose but were not used.
- **Duplicate student data store**: your attendance module keeps its own `attendance/students.json` registration list parallel to your teammate's student module, creating two diverging sources of truth for the same entity; and there are no automated tests anywhere in the repository to catch any of the above.

---

## Detailed Feedback (Instructor Review)

**Attribution note (team project):** You were graded only on work verifiably authored by you. Your 3 commits (all on a single day) produced the **attendance module full-stack** (`attendance_routes.py`, `attendance_service.py`, AttendanceDashboard/Manage pages, `attendanceApi.js`) and the **grades module backend** (`grade_routes.py`, `grade_service.py`). The grades frontend, auth, and student CRUD are your teammate's.

**What you did well**
- The attendance module is clean and genuinely full-stack: thread-safe JSON file writes (`threading.Lock()`), a service/routes split, docstrings, and a frontend that actually calls your API. That is real, working engineering.

**Where to grow**
- **No security at all.** Your routes have no `Depends()` guard, no token, no hashing — the API is wide open. Add the shared login's protection to your own routes.
- **Scope is narrow and incomplete.** Two modules, one of which (grades) you left half-finished by not building its frontend. Own features end-to-end.
- **All your work landed on one day.** That is not a development process — commit incrementally so your progress is visible and attributable.

I am aware the grades UI was handed to your teammate; in a team project, partial ownership of a feature limits what either of you can claim for it.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
