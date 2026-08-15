# Code Assessment Report

**Student:** Jotirmoy Mollick
**ID:** 2320133
**Section:** Section 5 (Group S5-19)
**Project:** Smart AI Warehouse Management System
**Project Type:** Individual
**GitHub:** https://github.com/Jotimoy/CSE309

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/main.py`, 5 routers, 17 routes)
- Frontend: ✅ React + TypeScript confirmed (Vite; `index.html` → `main.tsx`)

## Assessed at Commit
- **SHA:** `91b1be4` — 2026-08-13 — Merge pull request #23 (feature/database-model)
- **Repo:** 16 commits, 6 active days (2026-06-18 → 2026-08-13), single author (Jotirmoy Mollick)

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Token-based authentication | ✅ Implemented | Register/login, PBKDF2-hashed passwords, opaque server-side tokens (30-day TTL), `GET /users/me`. |
| Multi-location stock management | ✅ Implemented | `POST/GET /inventory/locations`, items CRUD, `POST /inventory/stock/set`, `POST /inventory/stock/adjust`. |
| Movement history | ✅ Implemented | `GET /inventory/movements` with stock-adjustment reasons. |
| Low-stock alerts & notifications | ✅ Implemented | Background scanner thread (60 s interval), `GET /inventory/alerts`, `POST /inventory/alerts/{id}/resolve`, threshold summary endpoint. |
| Responsive dashboard | ✅ Implemented | `Dashboard.tsx` with stat panels, `InventoryList.tsx`, `ItemDetail.tsx`, `AlertsPage.tsx`, SVG `Icons.tsx`. |
| AI-assisted capabilities (computer vision / OCR) | ❌ NOT implemented | Claimed in the project title/overview, but no AI dependency exists (`requirements.txt` has only fastapi/uvicorn/dotenv/pydantic/pytest) and no CV/OCR code found. Marketing name only. |
| Rate limiting / request logging | ✅ (bonus) | In-memory per-IP rate limiter middleware (env-configurable) + HTTP request logging middleware. |

## Security & Authentication
- Password hashing: ✅ strong — PBKDF2-HMAC-SHA256, 100k iterations, 16-byte per-user salt (`auth_service.py`), constant-time compare
- Token scheme: ✅ real opaque tokens stored server-side (`auth_tokens` table, `secrets.token_urlsafe(32)`) — not JWT
- ⚠️ **Major gap:** only `/users/me` and `/auth/me` require a token. The **entire inventory API is unguarded** — `inventory_router.py` and `dashboard_router.py` have zero `Depends`/auth checks, so any client can create items, adjust stock, resolve alerts, and read the summary without logging in.
- ⚠️ No RBAC enforcement: `role` column exists in the schema but no endpoint checks it.
- ⚠️ CORS middleware allows all origins (`allow_origins=["*"]`) despite the code comment saying production should tighten it via env.

## Data Persistence
- Storage: ✅ SQLite via raw `sqlite3` (`database.py`), tables: users, auth_tokens, locations, items, stock, stock_movements, alerts
- Frontend wiring: ✅ `services/api.ts` with `setAuthToken`/`fetchWithAuth`, `AuthContext.tsx` token flow, `RequireAuth.tsx` route guard, typed pages — no hardcoded data

## Runnability (tested in this session)
- Backend: ✅ boots (database auto-initialized at import); **5/5 tests pass** (`test_alerts.py`, `test_api_endpoints.py`, `test_inventory.py`) when run with `PYTHONPATH` set per the repo layout — matches the report's "all passing" claim
- Frontend: static review only

## Observations
- **Repo hygiene issues:** committed database files (`warehouse.db` ×3 at root/backend, `smart_todo.db`, `test_warehouse.db`), `.DS_Store`, `.vite/deps` cache, and stray duplicate entries (`App.jsx`/`App.tsx`, `main.jsx`/`main.tsx` — only the `.tsx` pair is used).
- Good infrastructure instincts: Docker + docker-compose, `.env.example`, `scripts/` (init_db, seed_demo, e2e_smoke), background-scanner design deliberately testable, `backend/main.py` as a clean re-export shim.
- Rate limiter and request logging are beyond-required touches.

## Future Scope
- Protect every `/inventory/*` and `/dashboard` route with `Depends(_get_current_user)` (or a router-level dependency) — this is the single most important fix.
- Remove committed DB files, `.DS_Store`, `.vite` cache, and the stray `.jsx` duplicates; add a `.gitignore`.
- Either implement or drop the "AI/OCR" claim — as submitted there is no AI component.
- Enforce the `role` column if admins are intended; tighten CORS via env.

---

## Additional Code-Review Findings

- **Token TTL is declared but never enforced**: `auth_service.py` defines `TOKEN_TTL_DAYS = 30`, yet `get_user_by_token()` only matches the token string and never compares `created_at` against the TTL — tokens are valid forever. There is also no logout/revocation endpoint (`auth_router.py` exposes only register/login/me; no `DELETE FROM auth_tokens` exists anywhere), so a leaked token can never be invalidated.
- **The passing test suite would not catch the biggest defect**: all 5 tests (`test_health`, `test_inventory_flow`, `test_alerts_flow`, DB-init and low-stock tests) exercise only unauthenticated happy paths — not a single test touches registration, login, or a 401/403 case. The unguarded `/inventory/*` API passes the entire suite, so "5/5 passing" currently certifies very little.
- **Rate limiter leaks memory and the error handler leaks internals** (`backend/app/main.py`): `app.state.rate_buckets` accumulates one entry per (IP, minute) tuple and is never purged — an unauthenticated attacker can grow server memory indefinitely just by rotating requests. The same middleware returns `{"detail": str(exc)}` on unhandled exceptions, exposing raw internal error text (including SQL errors) to clients. (`import time` also appears twice.)
- **Import-time side effects**: `backend/app/main.py` initializes the database and starts the low-stock scanner thread at module import (comments admit this is done "so tests that instantiate TestClient benefit"). Importing the module mutates the filesystem and spawns threads — configuration should happen in the lifespan handler with the scanner made injectable/disableable instead.
- **Registration is a user-enumeration oracle**: `POST /auth/register` returns `400 "A user with this email already exists"` while login uses the generic "Invalid email or password". Additionally, `create_user()` wraps all insert failures in a generic `RuntimeError`, so a duplicate-email race produces an opaque 500 rather than a clean 409.
- **`smart_todo.db` committed in `backend/`**: a SQLite database belonging to a different project (a todo app) is checked in alongside `warehouse.db` — evidence the repo was assembled by copying from older coursework. Remove it.

## Detailed Feedback (Instructor Review)

**What you did well**
- Strong password cryptography: PBKDF2-HMAC-SHA256 with 100k iterations and a per-user salt, with constant-time comparison. Opaque server-side tokens with a 30-day TTL are a legitimate alternative to JWT.
- Good infrastructure instincts: Docker + docker-compose, `.env.example`, seed/smoke scripts, a background low-stock scanner that is deliberately testable, and 5/5 passing tests. The rate limiter and request-logging middleware are beyond-required touches.

**Where to grow (the critical gap is route protection)**
- You built real auth and then protected almost nothing with it. Only `/users/me` and `/auth/me` require a token — the **entire inventory and dashboard API is unguarded**, so any client can create items, adjust stock, and resolve alerts without logging in. Authentication that isn't enforced on your data routes provides no real security. Adding a router-level `Depends(get_current_user)` to `inventory_router` and `dashboard_router` is the single highest-value fix.
- **Repo hygiene:** you committed database files (`warehouse.db`, `test_warehouse.db`), `.DS_Store`, a `.vite` cache, and stray duplicate `.jsx`/`.tsx` files. Add a `.gitignore` and remove these.
- The project title advertises "AI / OCR" but no such dependency or code exists. Either build it or rename the project — an unmet headline claim undermines an otherwise honest report.

**Submission note**
- The submitted repo was a shallow clone (1 local commit), but your full history was confirmed via the GitHub API: 16 commits across 6 active days (2026-06-18 → 2026-08-13). Your development process is genuine and well-spread. In future, submit the full `.git` history so this is immediately visible.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
