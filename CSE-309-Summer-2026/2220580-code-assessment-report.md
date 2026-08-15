# Code Assessment Report

**Student:** Sabbir Ahmmed
**ID:** 2220580
**Section:** Section 5
**Project:** Smart Task Manager — Personal Task Tracking App
**Project Type:** Individual
**GitHub:** https://github.com/Sabbir-0580/smart-task-manager

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app/main.py`, 1 router: task_routes)
- Frontend: ✅ React confirmed (`"react": "^19.2.8"` in `package.json`, JSX components)

## Assessed at Commit
- **SHA:** `3b5668261c3a129af798fba045dfccbfa787a830`
- **Date:** 2026-08-12
- **Message:** "Point frontend to live Render backend URL"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 44 (verified via GitHub API) |
| First Commit | 2026-07-01 |
| Last Commit | 2026-08-12 |
| Active Days | 6 |
| Contributors | 1 (Sabbir-0580) |

## Features Claimed vs Found

| Feature | Status | Notes |
|---------|--------|-------|
| Task creation | ✅ Implemented | `POST /tasks` — creates a task with title, description, due_date, completed, category fields. |
| Task listing | ✅ Implemented | `GET /tasks` — returns all tasks from SQLite database. |
| Task detail | ✅ Implemented | `GET /tasks/{id}` — returns single task by ID. |
| Task update | ✅ Implemented | `PUT /tasks/{id}` — updates all task fields. |
| Task deletion | ✅ Implemented | `DELETE /tasks/{id}` — removes a task. |
| User authentication / login | ❌ Not found | No user accounts, no login, no auth at all. All tasks are shared among anyone with API access. |
| Role-based access | ❌ Not found | No concept of users or permissions. |

## Security & Authentication
- Password hashing: ❌ Not implemented — no user system exists.
- Token type: ❌ No authentication system of any kind.
- Protected routes: ❌ All 5 task endpoints are fully public.
- Role enforcement: ❌ Not applicable — no users.

## Data Persistence
- Storage method: ✅ SQLAlchemy ORM with SQLite database. Tables created on startup via `Base.metadata.create_all()`. Task model with proper fields (title, description, due_date, completed, category).
- Frontend-backend integration: ✅ Fully wired — `taskService.js` uses axios to call all 5 backend endpoints. Frontend deployed on Vercel, backend deployed on Render.

## Runnability
- Backend: ✅ Started cleanly — HTTP 200, "Smart Task Manager API is running".
- Frontend: ✅ Started successfully — Vite v8.2.0, HTTP 200, 1430ms.
- Deployment: ✅ CORS config includes `allow_origin_regex=r"https://.*\.vercel\.app"` — evidence of real deployment.

## Observations
- **Working, deployed full-stack app**: The backend and frontend both run cleanly and are wired together. This is a functional product.
- **Very narrow scope**: The project is limited to a single entity (tasks) with basic CRUD. There is no user system, no authentication, no multi-user support, and no additional features.
- **Well-commented code**: The backend code includes comments explaining each function's purpose — good learning habit.
- **Deployed to Render + Vercel**: Commendable initiative to host the application publicly.
- **No auth** is the critical gap. Without a user system, all users share the same task list — the app has no concept of "whose tasks these are."

## Future Scope
- Add user registration and login: a `POST /register` and `POST /login` endpoint returning a JWT would allow personal task lists per user.
- Once auth exists, add `user_id` as a foreign key on the `Task` model and filter `GET /tasks` by the logged-in user.
- Expand the feature set: task categories with filtering, due date reminders, priority levels, or subtasks would significantly improve the scope.
- Continue the habit of deploying — it shows initiative and real-world engineering awareness.

## Additional Code-Review Findings

- **A `.env` file is committed to git despite your own ignore rule.** `backend/.env` is tracked in the repository even though `.gitignore` contains `.env` — once a file is committed, the ignore rule stops applying to it. The file happens to be empty, so no secret leaked this time, but the pattern is dangerous: the next `DATABASE_URL` or API key you add locally would silently be pushed to GitHub. Run `git rm --cached backend/.env` to untrack it.
- **Dead configuration plumbing.** `backend/app/config.py` is an empty file and `requirements.txt` installs `python-dotenv`, yet nothing in the code loads or reads environment-based settings — the database URL in `backend/app/database/db.py` is hardcoded. Either wire the config module up or delete it; empty scaffolding suggests a feature that was started and abandoned.
- **No automated tests at all.** There is no `tests/` folder, no `test_*.py`, and `pytest` is not in `requirements.txt`. Even two or three `TestClient` tests (create → list → delete) would have caught regressions and demonstrated the testing practices from the course.
- **`GET /tasks` loads the entire table into memory.** `backend/app/routes/task_routes.py` uses `db.query(Task).all()` with no pagination or limit, and the schema places no length bound on `title`/`description` (`backend/app/schemas/task_schema.py`). A single huge task list — or one client posting very large payloads — would degrade the deployed Render instance for every user, since the data is shared globally.
- **The "update" endpoint silently discards partial-update semantics.** `PUT /tasks/{id}` requires a full `TaskCreate` body and overwrites every field, so a client trying to toggle `completed` must resend title, description, due date, and category — any omitted field is reset to its default. A `PATCH` with a `TaskUpdate` schema (all fields `Optional`) is the standard fix.
- **Positive notes:** the `get_db()` dependency in `task_routes.py` correctly closes the session in a `finally` block (many beginners leak sessions), `404` handling is consistent across read/update/delete, and `.gitignore` does properly exclude `venv/` and `*.db` — the `tasks.db` and `.venv` present in the submission folder are local artifacts, not committed files.
- **Minor:** `Task(**task.dict())` uses Pydantic v1 style; with `pydantic==2.13.4` installed, `task.model_dump()` is the current API.

## Detailed Feedback (Instructor Review)

**What you did well.** You shipped a genuinely working, deployed full-stack application. All five task CRUD endpoints are implemented cleanly over SQLAlchemy/SQLite, the React frontend is fully wired through an axios service layer, and the CORS configuration plus Render + Vercel deployment show you took the project beyond localhost. Your backend code is well-commented, which is a good habit.

**Where to grow.** Be honest with yourself about scope: this is one entity with five endpoints. The report shows no authentication at all — no users, no login, no tokens — so every endpoint is public and every visitor shares the same task list. For a task manager, that defeats the purpose of the product. The repository also contains only a single commit, so I cannot see any development process; the instructor is aware the git history does not demonstrate sustained work. Both gaps — no auth and minimal scope — were called out in the report and are the difference between a demo and a project.

**Submission note.** The submitted repo was a shallow clone (1 local commit), but your full history was confirmed via the GitHub API: 44 commits across 6 active days from July to August. Your development process is genuine and well-spread.

**future scope ideas:** add register/login with JWT and a `user_id` foreign key on tasks, then add filtering, priorities, or reminders to widen the feature set.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
