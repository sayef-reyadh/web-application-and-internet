# Code Assessment Report

**Student:** Md Sumon Islam
**ID:** 2221333
**Section:** Section 6
**Project:** MeatTech — Smart Inventory & Supply Chain
**Project Type:** Team (with SM Nafi 2220947)
**GitHub:** https://github.com/md-sumon-islam/MeatTech

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/main.py` + `New folder/.../backend/main.py`)
- Frontend: ✅ React confirmed (Vite, `frontend/` + `New folder/.../frontend/`)

## Assessed at Commit
- **SHA:** `693a386` — 2026-08-13 (repo HEAD)
- **Repo:** 30 commits, **10 active days** (2026-06-18 → 2026-08-13)
- **This student (`md-sumon-islam`):** 06-18 docs phase 1–2; 07-01 package.json init + Django requirements; 07-22 review API endpoint (`83837a9`) + ProductReview component (`0be1864`); **08-01 final upload** (`abd61b7` — latest FastAPI/React codebase moved to repo root); plus merges
- **Teammate (Nafi):** 07-02 base setup, 07-22 FastAPI+models, 07-30 frontend/backend code + UI styling CRUD, 08-06 conflict resolution + login/logout/dashboard, docs phase 3–4

## Features Found
- Reviews: GET/POST/PUT/DELETE `/api/reviews` (Pydantic schema, id counter)
- Users: GET/POST/PUT/DELETE `/api/users` (SQLite via raw sqlite3)
- Frontend: `ProductReview.jsx` (rating submission), `Dashboard.jsx`, `App.jsx` Vite app
- ⚠️ Repo contains a **duplicate nested tree** (`New folder/MeatTech-sumon-doc-phase1/...`) holding the actually-developed app (reviews + dashboard + login/logout UI), while repo root holds a separate simplified user-CRUD upload

## Security & Authentication (verified)
- ❌ **None.** No password hashing, no tokens, no `Depends`/auth guards on any endpoint (both backend copies)
- ❌ Nafi's "login/logout" change was frontend-UI-only (Dashboard.jsx) — no authenticated API exists
- ❌ CORS `allow_origins=["*"]` in both backends

## Data Persistence
- ⚠️ Primary review feature is an **in-memory Python list** (`reviews_db: List[Review] = []`) — lost on restart
- Root user CRUD uses a real SQLite file (`meattech.db`) via raw sqlite3 — persists, but trivial scope

## Runnability
- ⚠️ No tests present; `__pycache__` (pyc files) committed into git

## Observations
- Messy repository state: duplicate project trees nested inside `New folder/`, committed `__pycache__/*.pyc`, stray `backend/views.py` → removed, Django requirements mixed with FastAPI code, root-level upload appears to be a simplified demo while the "real" app lives in the nested folder.

## Future Scope
- Clean the repo (remove nested duplicates, __pycache__, stray files); move the review store to a real DB; add hashing + JWT + guards; single coherent backend.

---

## Additional Code-Review Findings

- **Your `ProductReview` component does not match your own API.** `frontend/components/ProductReview.jsx` POSTs `{ product_id, rating, comment }` to `/api/reviews/`, but the backend's `ReviewCreate` schema (`backend/main.py` review tree) requires `{ name, rating, comment }` and has no `product_id` field — every submission from your component fails with 422. The two halves of your feature were never integrated end-to-end.
- **Blocking I/O inside `async` endpoints.** All four handlers in `backend/main.py` are declared `async def` yet call the synchronous `sqlite3` driver directly — every request blocks the event loop for all other clients. Use sync `def` (thread pool) or an async driver.
- **Leaked connections on error.** Each endpoint opens `sqlite3.connect("meattech.db")` with no `try/finally` — any exception mid-handler leaves the connection (and possibly a lock on the DB file) open.
- **No uniqueness or format validation on email.** The `users` table has no `UNIQUE` constraint, and `UserSchema.email` is a plain `str` (not `EmailStr`) — duplicate and malformed emails are silently accepted.
- **Hardcoded backend origin.** `frontend/main.jsx` calls `http://127.0.0.1:8000/...` literally — there is no environment-based API base URL, so the frontend cannot be pointed at a deployed backend without editing source.
- **Undeclared runtime dependencies.** `backend/requirements.txt` lists only Django/DRF packages while `main.py` imports `fastapi` and `uvicorn` — installing from your own requirements file cannot run your app.

---

## Detailed Feedback (Instructor Review)

**What you did well.** You shipped working full-stack code: a FastAPI user CRUD that genuinely persists to SQLite via raw sqlite3, plus a reviews API with a proper Pydantic schema and a React `ProductReview` component for submitting ratings. Your commits touch both backend and frontend, which is more end-to-end integration than many students manage.

**Where to grow.** Bluntly: your contribution has no security at all. No password hashing, no tokens, not one auth guard on any endpoint — the login UI was your teammate's work and was frontend-only. Your main review feature stores data in an in-memory Python list, so every restart wipes it. Repository hygiene is poor: a duplicate nested project tree inside `New folder/`, committed `__pycache__` bytecode, Django requirements mixed into a FastAPI app, and wildcard CORS. This is not a submittable repository state.

**Attribution note.** This was a team project with SM Nafi. Git history credits you with the review API, the ProductReview component, and the root user-CRUD upload; base setup and the login/logout UI were Nafi's. You are assessed only on your own code.

**future scope ideas:** consolidate to one backend, persist reviews in a real database via an ORM, add bcrypt + JWT + route guards, and clean the repository.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
