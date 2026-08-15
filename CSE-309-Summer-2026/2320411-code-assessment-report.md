# Code Assessment Report

**Student:** Mafijur Rahman Hridoy
**ID:** 2320411
**Section:** Section 6
**Project:** Campus Management System
**Project Type:** Team (with Md. Mahamudul Amin 2320132)
**GitHub:** https://github.com/MahamudulAmin/campus-management-system

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/`)
- Frontend: ✅ React confirmed (TypeScript + Vite, `Frontend/src/`)

## Assessed at Commit
- **SHA:** `f35d9b6` — 2026-08-12 (repo HEAD)
- **Repo:** 44 commits, 13 active days
- **This student (`hridoy3126`):** 06-18/19 docs (SRS, TDD); **07-01 backend base setup** (`da1d8f8` — main.py, app.py, routes for complaints/offices/requests/user_routes, data files, requirements, Procfile); **07-30 Teacher Dashboard feature** (`1b2d403` — Login.tsx +155, Communications/Requests/Updates components, App.tsx restructure, backend app.py overhaul) + merge/conflict resolution (`5c89f28` — api.ts, vercel.json, main.py +439, page updates); plus PR merges

## Features Found (his contributions)
- Backend base: main.py/app.py, routes (complaints, offices, requests, user_routes) over JSON data files
- Frontend: Login page, Communications, Requests, Updates components, api.ts client, admin/student page updates
- Documentation: SRS + Technical Design Document

## Security & Authentication (verified)
- ❌ **None.** Login is ID-as-password (plaintext `login.json`), no tokens, no route guards, CORS `*` — same as team repo

## Data Persistence
- ✅ Runtime JSON file storage (his routes + data files) — real file-system persistence layer for the backend he built

## Runnability
- ⚠️ No tests

## Observations
- Second-biggest contributor: owns the backend base setup and a major frontend feature branch; ~4 focused work days with large, real code changes (1,000+ lines in the teacher-dashboard branch).

## Future Scope
- Add authentication to the routes he authored; keep per-feature commits with clear authorship; add tests for his route modules.

## Additional Code-Review Findings

- **Your request endpoint accepts unvalidated input.** `create_request(request: dict)` in `backend/routes/requests.py` takes a raw dictionary with no Pydantic schema — any keys a client sends (including a pre-set `"status"`) are written verbatim into `requests.json`. This is a mass-assignment hole in a route you authored.
- **ID minting is racy.** Request IDs (`REQ-####` in `requests.py`, the same pattern for complaint IDs in `complaints.py`) are computed by scanning every record and then rewriting the whole file, with no file locking — two concurrent submissions can receive the same ID or one write can silently overwrite the other.
- **Your base commit shipped a second, dead stack.** `backend/app.py` — included in your backend base setup — is a parallel Flask + MongoDB + JWT application with a hardcoded secret that the live FastAPI app never imports; its `token_required` decorator never executes, so the "auth" visible in your base setup is decorative.
- **The `/offices` prefix is served by two mounted routers** (`office_routes.py` and `offices.py`, both from your base commit) with overlapping CRUD and different path parameters (`{office_id}` vs `{id}`) — ambiguous ownership of the same resource that should be consolidated into one module.
- **List endpoints return entire files unpaginated and unfiltered** (e.g. `get_requests` returns every request record on each call), so reads scale with total data size and expose all users' records to any caller.

## Detailed Feedback (Instructor Review)

**What you did well.** Your slices of the campus system are real and substantial: the backend base setup, the teacher dashboard feature branch (roughly a thousand lines), and the SRS and technical design documents. The git history shows work on five distinct dates, which indicates sustained engagement rather than a single end-of-term push, and your merge/conflict-resolution commits show you can integrate others' work.

**Where to grow.** Scope and depth are the problem. Your contributions are a minority of a project that, as submitted, has no authentication anywhere — no guard, token, or authorization check in any of the eight route files, including the ones you authored. The backend base you set up persists to JSON files, which is a demo shortcut, not a database, and there are no tests for your modules. Owning "the backend base" means owning its architectural weaknesses; you cannot claim the setup while disclaiming its lack of auth and real storage.

**Attribution note.** Team project with Md. Mahamudul Amin. Your commits are clearly identifiable, but partial feature ownership of an insecure system still counts toward your assessment.

**future scope ideas:** add real authentication to the routes you authored, replace the JSON layer with SQLite or PostgreSQL, and write tests for your route modules.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
