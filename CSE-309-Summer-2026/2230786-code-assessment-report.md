# Code Assessment Report

**Student:** Arnima Rahman Annu
**ID:** 2230786
**Section:** Section 6
**Project:** AgriFlowTrack — Agricultural Inventory Management
**Project Type:** Individual
**GitHub:** https://github.com/ArniT786/AgriFlowTrack

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/`)
- Frontend: ✅ React confirmed (Vite, `frontend/src/`)

## Assessed at Commit
- **SHA:** `7e4e73c` — 2026-07-28 — "feat: Migrate codebase to FastAPI Backend and React Frontend"
- **Repo:** 22 commits, **6 active days** (2026-06-13 → 2026-07-28)

## Features Found
- Login screen + dashboard (counts + latest market data)
- 9 CRUD domains: Agricultural Products, Agri Inputs, Harvested Crops, Inventory, Market Data, Perishable Products, Post-Harvest Monitor, Storage Conditions, Tracking of Products (~41 endpoints, each with get/create/update/delete)
- Frontend: generic `CrudPage` component rendering each entity, Sidebar navigation, Dashboard, Login; fully wired to API

## Security & Authentication (verified)
- ❌ **No security at all.** `/api/login` compares plaintext credentials directly: `AdminUser.username == request.username AND password == request.password` — seeded admin is **`admin` / `admin123`** (hardcoded in `main.py`)
- ❌ Login returns `{success, username}` — **no token issued**, nothing is authenticated server-side
- ❌ **Every** CRUD endpoint has no `Depends`, no auth — all routes open to anyone
- ❌ CORS `allow_origins=["*"]`
- ⚠️ Data includes real-world placeholders (Mombasa/Nairobi markets) indicating template-based seed data

## Data Persistence
- ✅ SQLAlchemy + SQLite (env-overridable `DATABASE_URL`), 10 models, startup seeding

## Runnability
- ⚠️ Not testable via pytest — **no tests present** in repo

## Observations
- Functional but entirely unguarded admin-style CRUD app; the login screen is cosmetic (nothing on the API is protected). Single 546-line `main.py` with repetitive per-entity blocks; no tests, no migrations.

## Future Scope
- Replace plaintext login with hashed passwords (bcrypt/PBKDF2) + JWT + `Depends(get_current_user)` on every route; seed admin must use a hashed password; restrict CORS; split routes into modules; add tests.

---

## Additional Code-Review Findings

- **Non-RESTful resource addressing.** Single-record reads use a query parameter (`GET /api/agricultural_products?id=3`) and deletes pass the id in a JSON body (`DeleteRequest` on `DELETE`) instead of path parameters (`/api/.../{id}`). DELETE request bodies are dropped or rejected by many clients, proxies, and servers, so your delete flow is fragile by design.
- **Create/update responses discard the resource.** Every POST/PUT returns only `{"message": "..."}` — never the created record or its new id — forcing the frontend to refetch the whole list after each write. Updates are also full-object replacement only; there is no partial-update path.
- **`normalize_date` silently swallows bad input.** In `backend/schemas.py`, an unparseable date string is converted to `None` instead of raising a validation error — invalid client data is stored as NULL rather than rejected with 422, quietly corrupting data quality.
- **The "session" is a forgeable flag.** `frontend/src/components/Login.jsx` stores `localStorage.setItem('agriflow_auth', 'true')` after login — authentication state is a client-side boolean anyone can set in devtools, and the backend never sees or validates it.
- **Dashboard ordering bug.** `/api/dashboard` returns "latest" market data ordered by `id` descending rather than by the record's `date` — any out-of-order insert shows stale records as the latest prices.
- **Deprecated startup hook with swallowed errors.** `@app.on_event("startup")` is deprecated in current FastAPI (use `lifespan`), and the seeding block wraps everything in a broad `except` that only prints — a failed seed looks exactly like a successful one.

---

## Detailed Feedback (Instructor Review)

**What you did well**
- Impressive domain breadth: 9 CRUD domains (~41 endpoints) covering a coherent agri-inventory scope, backed by SQLAlchemy + SQLite with 10 models and startup seeding.
- The generic `CrudPage` React component driving every entity is a smart, DRY design — you wrote the CRUD UI once and reused it, which is good engineering.
- The frontend is genuinely wired to the backend (no hardcoded data arrays).

**Where to grow (the critical gap is that auth is cosmetic)**
- Your `/api/login` does check credentials against the database — that is real, and it is why this is scored as "weak auth" rather than "no auth." But it compares **plaintext passwords**, the seeded admin is literally `admin`/`admin123`, and **no token is issued**, so after login nothing on the API is actually protected. Every one of your 41 endpoints is open to anyone who skips the login screen entirely. The login form gates only the React UI, not the data.
- Concretely: hash the seeded password, return a signed JWT on login, and add `Depends(get_current_user)` to every CRUD route. Right now your login screen is a lock on the front door with the back wall missing.
- **Code structure:** a single 546-line `main.py` with repetitive per-entity blocks should be split into routers/modules (your generic CrudPage shows you already think in abstractions — apply the same idea to the backend).
- Add tests and restrict CORS from `*`.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
