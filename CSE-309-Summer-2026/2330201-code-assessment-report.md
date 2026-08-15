# Code Assessment Report

**Student:** Nujat-E- Hasnat
**ID:** 2330201
**Section:** Section 5
**Project:** Personal Expense Tracker
**Project Type:** Individual
**GitHub:** https://github.com/Nujat11/web_development_project

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `Backend/main.py`, routers registered via `APIRouter`)
- Frontend: ✅ React confirmed (`"react": "^18.3.1"` in `Frontend/package.json`, JSX components with hooks)

## Assessed at Commit
- **SHA:** `bba42e7`
- **Date:** 2026-08-13
- **Message:** "style: remove seed mock data reference from empty state message"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 137 |
| First Commit | 2026-06-11 |
| Last Commit | 2026-08-13 |
| Active Days | 12 |
| Contributors | 1 (Nujat-E- Hasnat) |

> Commit history retrieved from the GitHub API (the submitted folder contains a code snapshot without `.git`). All 137 commits authored by the student alone, consistent with an individual project. Commit messages follow a conventional style (`feat:`, `style:`, `docs:`), and feature branches (`#01-Frontend`, `#02-Backend`) were used with merges into `main`.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| User registration and login | ✅ Implemented | `POST /register`, `POST /login` in `routes/user_routes.py`; `Login.jsx` / `Register.jsx` pages. Passwords hashed with bcrypt. |
| Full CRUD for income & expense transactions | ✅ Implemented | `POST/GET/PUT/DELETE /expenses` in `routes/expense_routes.py`; wired in `Dashboard.jsx` via modal add/edit and inline delete. |
| Dashboard summary (balance, income, expenses) | ✅ Implemented | Stat cards computed from live transaction data in `Dashboard.jsx`. |
| Category-wise spending pie chart (Recharts) | ✅ Implemented | `ExpenseChart.jsx` — groups expenses by category, renders `PieChart` with legend, tooltip, empty-state handling. |
| Budget progress bars | ✅ Implemented | Monthly spending limit with editable limit, color-coded progress bar, and limit-exceeded indicator. |
| Wallet management (multi-wallet) | ✅ Implemented | `POST/GET/DELETE /wallets` in `routes/wallet_routes.py`; `Main Wallet` protected from deletion; deleting a wallet cascades to its transactions. |
| MongoDB Atlas persistence with sequential IDs | ✅ Implemented | `database.py` uses PyMongo with an atomic `counters` collection for integer IDs; all controllers read/write MongoDB. |
| Production deployment (Netlify + Render) | ✅ Implemented | `render.yaml`, `netlify.toml`, `vercel.json`; live backend URL configured in `api.js` with CORS enabled. |

## Security & Authentication
- Password hashing: ✅ bcrypt via direct `bcrypt` library (`user_controller.py` uses `bcrypt.hashpw` with generated salt)
- Token type: ❌ No token issued — login returns the user record; there is no JWT/session token of any kind
- Protected routes: ❌ No backend route protection — no `Depends()`, no auth guard on any endpoint (any user's expenses are readable via `/expenses/{user_id}`)
- Role enforcement: ❌ No roles / RBAC
- Session protection: ⚠️ Frontend-only — `localStorage` user record; `Dashboard.jsx` redirects to `/login` if absent, which is not enforced server-side

## Data Persistence
- Storage method: ✅ MongoDB (Atlas) via PyMongo — `users`, `expenses`, `wallets` collections plus `counters` for sequential IDs
- Frontend-backend integration: ✅ Fully wired in "Online" mode — `dataService.js` calls all five API endpoints through axios (`api.js`); deployed backend URL default with localhost fallback
- Note: The frontend also includes an "Offline" mode that stores data in `localStorage` (with optional seed demo data). The app defaults to Offline unless toggled, but the Online path is complete and functional.

## Runnability
- Backend: ✅ Started successfully — `uvicorn main:app` → HTTP 200 on `/health`. (Register/login require a live MongoDB, per normal setup.)
- Frontend: ✅ Built successfully — `npm install` (329 packages) + `npm run build` → `✓ built in 10.83s` (one chunk-size warning only).
- API wiring: ✅ Frontend calls backend via axios for login, register, expenses, and wallets.

## Observations
- **Strong feature scope and depth**: every feature claimed in the report is implemented with real logic — 6+ distinct working features (auth, transaction CRUD, dashboards, charting, budgets, multi-wallet).
- **Clean MVC-style backend**: clear separation of `routes` / `controllers` / `schemas`; consistent Pydantic validation; graceful HTTP error responses (404/400).
- **Good engineering process**: 137 commits across 12 active days spanning two months, conventional commit messages, feature-branch workflow, and comprehensive course documentation phases committed alongside code.
- **No secrets committed**: only `.env.example` files present; `.gitignore` in place.
- **Main gap — security**: authentication is real (bcrypt hashing) but session-less. There is no token, and every API endpoint is unauthenticated, so the "private, secure account" described in the report is not enforced server-side. Frontend-only redirects are bypassable.
- Budget limits and the offline-mode demo data are stored client-side only (acceptable as UI features, but they do not persist per-user in the backend).

## Future Scope
- Add a real token flow: issue a signed JWT on login (e.g., PyJWT with `exp`), and protect expense/wallet routes with `Depends(get_current_user)` using the user ID from the token instead of trusting a client-supplied `user_id`.
- Enforce ownership checks so users can only read/write their own data, and return 401/403 appropriately.
- Persist the budget limit server-side so it survives across devices and browser sessions.
- Consider making "Online" mode the default on the deployed site so the app always uses the real database.

## Additional Code-Review Findings

- **Schemas validate almost nothing**: `schemas/expense.py` types `type` as a free-form `str` (no `Literal["income","expense"]` or enum), so any string is accepted even though dashboard totals and the pie chart depend on it; `amount: float` has no constraint, so negative or zero "expenses" silently corrupt the budget math. `schemas/user.py` uses plain `str` for email — no `EmailStr`, no password length rule.
- **Duplicate accounts via email casing**: `get_user_by_email` (`controllers/user_controller.py`) matches the email string exactly with no `.lower()` normalization — `A@b.com` and `a@b.com` register as two different accounts, and login then only works for the exact casing used at registration.
- **Wallet integrity is fragile**: wallets are keyed by a name string rather than an ID (`controllers/wallet_controller.py`), `create_wallet` performs no duplicate-name check, and `delete_wallet` cascades by name (`delete_many({"user_id":..., "wallet":...})`) — two same-named wallets would both lose their transactions in one delete. Expenses can also reference wallet names that do not exist.
- **CORS is wide open**: `main.py` sets `allow_origins=["*"]` together with `allow_credentials=True` — an invalid pairing that browsers partially reject; enumerate the Netlify frontend origin explicitly.
- **Response-code and enumeration issues**: failed login returns 400 (should be 401) while `POST /register` returns `400 "Email already registered"` — a user-enumeration oracle. (Credit where due: the `UserOut` response model correctly strips the bcrypt hash from login/register responses.)
- **Zero automated tests** — the repo has no test files at all. Positive note: the Mongo layer shows real care — atomic sequential IDs via `find_one_and_update` with `upsert`, consistent `_id` exclusion in projections, and `exclude_unset=True` on partial updates are all correct patterns.

## Detailed Feedback (Instructor Review)

**What you did well.** The feature scope is genuinely strong: transaction CRUD, a live dashboard, category pie charts, budget progress, and multi-wallet management with cascade deletes — all wired to a real MongoDB backend with sequential IDs, a clean routes/controllers/schemas layout, deployment configs for Netlify and Render, and 137 commits with conventional messages. Bcrypt password hashing is implemented correctly.

**Where to grow.** I have to be blunt: the security story stops at hashing. Login returns the user record and issues no token of any kind. No endpoint uses `Depends`, no JWT library appears in requirements, and expenses are fetched by a raw client-supplied `user_id` — meaning anyone can read or modify any user's financial data simply by changing a number in the URL. The frontend `localStorage` redirect is not authentication; it is decoration. The "private, secure account" your report promises does not exist server-side, and for a finance app this is a serious flaw, not a cosmetic one. Budget limits also live only in the browser.

**Submission note.** The submitted folder is a snapshot without `.git`; authorship was verified via the GitHub API.

**future scope ideas:** issue signed JWTs at login, protect every route with `get_current_user`, derive the user from the token — never from the client — and persist budgets server-side.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
