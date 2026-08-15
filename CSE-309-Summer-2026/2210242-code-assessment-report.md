# Code Assessment Report

**Student:** Anni Rahman Tahrim Tabassum
**ID:** 2210242
**Section:** Section 5
**Project:** FashionPanda — Express Fashion Delivery Platform
**Project Type:** Individual
**GitHub:** https://github.com/Anni-Rahman4418/FashionPanda

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app/main.py`, 5 routers: auth, products, orders, payments, users)
- Frontend: ✅ React confirmed (`"react": "^18.3.1"` in `package.json`, TypeScript/TSX components)

## Assessed at Commit
- **SHA:** `9293cdf73366c962f0ac5d350a58830263dfb250`
- **Date:** 2026-08-13
- **Message:** "Delete frontend/Login.jsx"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 1 |
| First Commit | 2026-08-13 |
| Last Commit | 2026-08-13 |
| Active Days | 1 |
| Contributors | 1 (Anni-Rahman4418) |

> **Note:** Only 1 commit exists in the submitted repository. The commit message ("Delete frontend/Login.jsx") suggests a prior history exists elsewhere — but no verifiable commit history is available from this submission.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Product listing / search | ✅ Implemented | `GET /api/products` backend route with category and search query filters. `CustomerMarketplace.tsx` frontend page. |
| Product management (admin) | ✅ Implemented | `POST/PUT/DELETE /api/products` routes. `AdminManager.tsx` frontend page. `ProductFormModal.tsx` for CRUD. |
| User registration / login | ✅ Implemented | `POST /api/register` and `POST /api/login` backend routes. Login compares credentials in SQLite. |
| Shopping cart | ✅ Implemented | `CartDrawer.tsx` component with full cart state management. |
| Checkout / orders | ✅ Implemented | `CheckoutModal.tsx` + `POST /api/orders` backend route. `OrderTrackerModal.tsx` for order status. |
| Payments | ✅ Implemented | `payments.py` router present with payment routes. |
| Retailer management | ✅ Implemented | `RetailerManager.tsx` frontend page. Users router handles retailer role. |
| Multi-role support | ✅ Implemented | Roles: `admin`, `retailer`, `customer` — set at registration and stored in SQLite users table. |

## Security & Authentication
- Password hashing: ❌ Plaintext — `INSERT INTO users (..., password, ...)` stores raw password string. Login uses `WHERE email = ? AND password = ?` directly in SQL.
- Token type: ❌ No token issued on login. The login endpoint returns a user dict (`{id, name, email, role}`) with no JWT or session token.
- Protected routes: ❌ All 5 routers have 0 `Depends()` calls — every endpoint is publicly accessible. Products can be added/deleted by anyone without logging in.
- Role enforcement: ❌ Roles exist in the database but are never checked server-side — no route guards enforce admin-only or retailer-only access.

## Data Persistence
- Storage method: ✅ SQLite — all entities (users, products, orders, payments) stored in `fashionpanda.db` via `sqlite3`. Schema created on startup via `init_db()`.
- Frontend-backend integration: ✅ **Backend-first with localStorage fallback** — `api.ts` calls the backend via axios first; on failure, falls back to localStorage (seeded from `mockData.ts`). Backend is the primary data source. This is a resilient offline pattern, not a hardcoded data approach.

## Runnability
- Backend: ⚠️ Entry point issue — uvicorn was invoked with `run:app` but `run.py` does not export `app` (it calls `uvicorn.run()` internally). Correct command is `uvicorn app.main:app`. Code itself is valid; this is a startup configuration issue.
- Frontend: ✅ Started successfully — Vite v5.4.21, HTTP 200, 1210ms.
- API wiring: ✅ Frontend has a working backend-first API layer — all CRUD operations attempt the backend before using localStorage.

## Observations
- **Well-structured project**: 5 backend routers with a clean router/schemas/serializers/database separation. TypeScript frontend with proper type definitions (`types/index.ts`).
- **Good offline resilience pattern**: The `apiService` approach with localStorage fallback shows thoughtful frontend architecture.
- **Authentication is the critical gap**: The login flow is structurally correct (POST credentials → get user back) but is missing the token step. Without a JWT returned and stored, subsequent requests carry no proof of identity.
- **1 commit in the submission** — it's unclear if this is a shallow clone or if most work was done locally without committing regularly.
- The last commit message ("Delete frontend/Login.jsx") at submission time suggests a separate Login component was removed, with auth integrated elsewhere.

## Future Scope
- Return a JWT token from the `/api/login` endpoint using `python-jose`. Store it in `localStorage` on the frontend and send it as `Authorization: Bearer <token>` on all subsequent requests.
- Hash passwords using `passlib[bcrypt]` before storing — never compare raw strings in SQL.
- Add `Depends(get_current_user)` to `products.py` write routes (POST/PUT/DELETE) and `orders.py` to enforce authentication server-side.
- Use `git` regularly — committing only once at submission makes it impossible to trace development progress.

---

## Additional Code-Review Findings

- **Privilege escalation at registration.** `UserRegisterSchema` in `app/schemas.py` accepts `role` straight from the request body, and `register_user` in `app/routers/auth.py` inserts it unchecked — anyone can sign up with `"role": "admin"`. Roles must be assigned server-side, never chosen by the client.
- **A default password in the schema.** `UserRegisterSchema` declares `password: str = "123456"` — any registration request that omits the password field creates an account protected by "123456". Never give credentials a default value.
- **Payments are fabricated.** `process_payment` in `app/routers/payments.py` accepts a client-supplied `amount` and unconditionally stores `status="Success"` — no gateway, no check that the amount matches the order. The inline comment ("payments are always 'Success' for now") is honest, but as built, any order can be marked paid for any amount by anyone.
- **Fragile ID generation.** User and payment IDs are derived from `SELECT COUNT(*) ... + 101` / `+ 501` — concurrent requests will collide on the same ID, and the IDs are trivially predictable. Use UUIDs or SQLite's AUTOINCREMENT.
- **Fairness note:** every SQL statement is parameterized with `?` placeholders — the plaintext-password problem is real, but there is no SQL-injection exposure, and that habit is worth keeping.
- **Zero automated tests** (no test files among the 63 tracked files). On the plus side, repo hygiene is clean — no `.venv`, `.db`, `__pycache__`, or `.env` is tracked — and the 20-file `docs/` set (SRS, ERD, DFD, PRD, API design) is thorough planning work.

## Detailed Feedback (Instructor Review)

**What you did well**
- Genuinely impressive scope: 5 routers (auth, products, orders, payments, users) covering marketplace, cart, checkout, order tracking, and multi-role support (admin/retailer/customer). Clean router/schemas/serializers separation and a typed TypeScript frontend.
- The `apiService` backend-first / localStorage-fallback pattern is a thoughtful, resilient design — you built an offline-tolerant data layer, not hardcoded data.

**Where to grow (the critical gap is security — it is effectively absent)**
- I need to be direct: your application has **no real security**. Passwords are stored and compared in **plaintext SQL** (`WHERE email = ? AND password = ?`), login returns **no token**, and not one of your 5 routers uses `Depends()` — so anyone can add or delete products and place orders without ever logging in. The roles you store (`admin`/`retailer`/`customer`) are never checked server-side. This is the single biggest thing holding the project back: you built a full multi-role marketplace and then left every door unlocked.
- The fix is well within your ability: hash passwords with bcrypt, return a signed JWT from `/api/login`, and add `Depends(get_current_user)` plus a role check to the write routes. Your frontend already tracks role — enforce it on the backend.

**Submission note**
- Only 1 commit was submitted, so I could not verify any development process. Commit regularly so your work is traceable.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
