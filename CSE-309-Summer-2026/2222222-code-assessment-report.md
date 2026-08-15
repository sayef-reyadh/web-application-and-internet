# Code Assessment Report

**Student:** Maruf Hossain Rafi
**ID:** 2222222
**Section:** Section 5 (Group S5-23)
**Project:** Personal Media Collection Manager
**Project Type:** Individual
**GitHub:** https://github.com/rafi-103/media-collection-manager

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/main.py` + `app/` package, 10 routes)
- Frontend: ✅ React confirmed (Vite, `src/App.jsx` + `services/api.js` via Axios)

## Assessed at Commit
- **SHA:** `28bde1f` — 2026-08-12 — "Fix: Sort items to show newest first and update gitignore"
- **Repo:** 78 commits, 6 active days (2026-06-18 → 2026-08-12), single author (Maruf Hossain Rafi)

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Media item CRUD (books & movies) | ✅ Implemented | `POST/GET/PUT/DELETE /api/items`, full attribute set (title, type, creator, genre, status, rating 1–5, platform, is_favorite, notes). |
| Search | ✅ Implemented | `GET /api/items/search/?keyword=` (title or creator). |
| Filter by type / status | ✅ Implemented | `/filter/type/?type_filter=book|movie`, `/filter/status/?status=`. |
| Dashboard statistics | ✅ Implemented | `/api/items/stats/` (total, books, movies, favorites) + stat cards in UI. |
| Responsive glassmorphism UI | ✅ Implemented | 736-line `App.jsx`, hover effects, newest-first ordering (latest commit). |
| User authentication | ❌ Not implemented | No users, no login/register, no tokens — the API is fully public. |

## Security & Authentication
- ❌ **No authentication at all**: no `users` table, no JWT, no session, no `Depends` auth guard anywhere. Every `/api/items` route is public (verified — the only `Depends` is `get_db`).
- CORS correctly restricted to `http://localhost:3000`.

## Data Persistence
- Storage: ✅ PostgreSQL via SQLAlchemy ORM (`items` table, `server_default=func.now()` timestamp)
- ⚠️ `DATABASE_URL` is hardcoded in `app/core/database.py` with a placeholder password (`postgres:YOUR_PASSWORD@localhost/media_collection_db`) — the backend cannot run as submitted without editing source (no `.env`/`.env.example`).
- Frontend wiring: ✅ Axios service layer (`services/api.js`), all CRUD/search/filter/stats flows call the API; no hardcoded arrays

## Runnability
- Backend: ✅ all 7 Python files pass `py_compile`
- Frontend: static review only

## Observations
- **Best-in-class backend structure seen so far**: clean layered architecture — `routes.py` → `services/item_service.py` → `repositories/item_repo.py` → SQLAlchemy models, plus Pydantic schemas — exactly the separation the rubric's quality criteria reward.
- Proper HTTP status codes (200/201/400/404/422), rating validation (1–5) in the service layer, `exclude_unset`-style partial updates.
- Report (6 pages) is detailed and honest; they actively cleaned `__pycache__` from git history and used GitHub Issues/Kanban/PRs (78 commits — but concentrated on only 6 active days).
- 22 documentation files (overview → PRD → SRS → TDD/ERD/API design).

## Future Scope
- This project needs authentication as its single highest-priority addition: user model + JWT/bcrypt + `Depends(get_current_user)` on the item routes, so a collection is per-user.
- Move `DATABASE_URL` (and secrets) to `.env` with a committed `.env.example`.

## Additional Code-Review Findings

- **Inconsistent error handling between create and update**: `PUT /api/items/{id}` wraps the service call in `try/except ValueError` and returns 400, but `POST /api/items/` in [routes.py](repo/backend/app/api/routes.py) calls the same rating-validating service with no handler — an out-of-range rating on create surfaces as an unhandled 500 instead of the 400 the update path produces.
- **Stats endpoint loads the whole table into memory**: `/api/items/stats/` calls `get_all_items_service()` and counts in Python list comprehensions instead of issuing SQL `COUNT` queries — response time and memory grow linearly with collection size for what should be four cheap aggregate queries.
- **Fragile route ordering**: `GET /api/items/{item_id}` is registered before `/search/`, `/filter/type/`, `/filter/status/`, and `/stats/`. It currently works only because FastAPI falls through when `"search"` fails integer parsing; any change to a string/UUID id scheme or careless reordering would silently shadow the search and stats routes.
- **Deprecated SQLAlchemy API**: [database.py](repo/backend/app/core/database.py) imports `declarative_base` from `sqlalchemy.ext.declarative`, which is deprecated under the pinned SQLAlchemy 2.0.23 — the 2.0-style import is `sqlalchemy.orm.declarative_base`.
- **Configuration dependencies declared but never used**: `requirements.txt` pins `python-dotenv` and `pydantic-settings`, yet neither is imported anywhere — the dependency list promises environment-based configuration that the code does not implement.
- **Zero automated tests and no migration story**: there is no `tests/` directory and no `pytest`/`httpx` in `requirements.txt`, so the rating validation and partial-update logic are entirely unverified; schema creation is `Base.metadata.create_all` at import time in [main.py](repo/backend/main.py) with no Alembic, so any model change requires manual database surgery.

## Detailed Feedback (Instructor Review)

**What you did well.** Your backend architecture is the cleanest seen in this cohort: a proper layered design (routes → service → repository → SQLAlchemy models) with Pydantic schemas, correct HTTP status codes, and rating validation in the service layer. Search, type/status filtering, and dashboard statistics are all implemented and wired through an axios service layer — no hardcoded frontend data. The 22 documentation files, use of GitHub Issues/Kanban/PRs, and cleanup of `__pycache__` from history show real process maturity.

**Where to grow.** The complete absence of authentication is a serious omission, and the instructor notes the gap between a "personal" collection manager and an API where the only `Depends` is `get_db` — anyone can read, edit, or delete everything. There is no user model, no token, no guard anywhere. Additionally, `DATABASE_URL` is hardcoded with a placeholder password, so the backend cannot run as submitted without editing source. Your 78 commits are concentrated in only 6 active days; steady pacing is part of the engineering discipline this course assesses.

**future scope ideas:** implement register/login with JWT and bcrypt, guard the item routes with `Depends(get_current_user)`, and move all connection strings into `.env` with a committed example file.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
