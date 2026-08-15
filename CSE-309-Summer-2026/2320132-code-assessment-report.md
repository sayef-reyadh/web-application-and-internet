# Code Assessment Report

**Student:** Md. Mahamudul Amin
**ID:** 2320132
**Section:** Section 6
**Project:** Campus Management System
**Project Type:** Team (with Mafijur Rahman Hridoy 2320411)
**GitHub:** https://github.com/MahamudulAmin/campus-management-system

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/`, 8 route modules + `main.py`)
- Frontend: ✅ React confirmed (TypeScript + Vite, `Frontend/src/`, ~19 pages/components)

## Assessed at Commit
- **SHA:** `f35d9b6` — 2026-08-12 (repo HEAD)
- **Repo:** 44 commits, **13 active days** (2026-06-16 → 2026-08-12)
- **This student (`MahamudulAmin`):** frontend + admin/student dashboards, login flow, requests/complaints, JSON data layer, deploy (Railway/Vercel), most commits
- **Teammate (Hridoy):** docs (SRS/TDD 06-19), backend base setup (07-01), Teacher Dashboard feature (07-30)

## Features Found
- Login (student/admin/staff/teacher roles via ID)
- Student: dashboard, submit service requests, complaints, notice board, university offices, profile, request history, notifications
- Admin: dashboard, users, offices, complaints, reports, activities
- Staff: service requests handling; Teacher: dashboard, announcements, messages
- 8 backend route modules backed by JSON data files

## Security & Authentication (verified)
- ❌ **Login is `password == student_id`** — code comment: "Currently password = student ID"; `login.json` stores **plaintext passwords** identical to IDs
- ❌ No tokens, no `Depends`/auth guards on any route — all JSON endpoints open
- ❌ CORS `allow_origins=["*"]`

## Data Persistence
- ✅ **Runtime JSON file storage** (`backend/data/*.json`: users, login, requests, complaints, offices, teacher messages/announcements, activities, updates, reports) — fully wired end-to-end with the React frontend

## Runnability
- ⚠️ No tests present; requires the JSON data folder (committed)

## Observations
- Broad, genuinely functional campus portal with rich frontend; the ID-as-password login is a demo shortcut with no real authentication. Deploy configs for Railway (Procfile) + Vercel included.

## Future Scope
- Real authentication: hashed passwords, tokens, and guards on every route; drop the "password = ID" behavior; restrict CORS; add tests.

## Additional Code-Review Findings

- **The login endpoint self-provisions accounts.** `POST /login` with any 7-digit ID that passes the regex creates the account on first use and sets its password to the ID (`backend/routes/login.py`). There is no enrollment step, and every login force-rewrites the stored record (`student["password"] = student_id`), so anyone can log in as any student ID at any time.
- **A single corrupt write wipes all credentials.** `read_login_users()` catches `JSONDecodeError` and silently resets `login.json` to an empty list — one truncated or concurrent write permanently destroys every stored credential and profile with no backup or recovery path.
- **No session or token is ever issued.** A successful login returns the profile object and the frontend persists it in `localStorage` (`Frontend/src/login.tsx`, `AdminSidebar.tsx`); every subsequent request is unauthenticated, so "logged in" is purely client-side state that any script can forge.
- **Two competing implementations of the staff request endpoints.** Inline handlers in `backend/main.py` are deliberately registered before `routes/staff.py` "so the main.py implementation handles teacher_requests.json directly" (the code comment says exactly this) — behavior silently depends on registration order, and the two paths can diverge.
- **Duplicate routers on the same prefix.** Both `backend/routes/office_routes.py` and `backend/routes/offices.py` mount CRUD on `/offices` with inconsistent parameter names (`{office_id}` vs `{id}`); the first registered match wins, making the effective API ambiguous.

## Detailed Feedback (Instructor Review)

**What you did well.** You built a broad, genuinely functional campus portal: student, admin, staff, and teacher flows, service requests, complaints, notices, and a JSON-backed data layer wired end-to-end into a React frontend. The feature breadth and the deployment setup (Railway/Vercel) show real full-stack effort.

**Where to grow.** Bluntly: this project has no security at all. Login compares a plaintext password against the student ID, and the credentials file stores them in plaintext. Every route is unauthenticated — not one of the eight route modules uses a dependency guard, token check, or authorization header. The JWT code in `backend/app.py` is dead Flask code that never runs; the app serves from `main.py`. Thirteen JSON files with runtime read/write is persistence, but it is not a database and will corrupt under concurrent writes. CORS is wide open. This is a serious omission, not a polish item.

**Attribution note.** Team project with Mafijur Rahman Hridoy. You own the majority of commits and the frontend; the missing-auth problem spans code you both touched, so responsibility is shared.

**future scope ideas:** hash passwords with bcrypt, issue JWTs, guard every route, replace JSON storage with a real database, restrict CORS, add tests.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
