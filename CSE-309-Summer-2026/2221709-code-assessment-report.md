# Code Assessment Report

**Student:** Jannatul Mahia
**ID:** 2221709
**Section:** Section 5
**Project:** Expense Tracker — Personal Finance Management App
**Project Type:** Individual
**GitHub:** https://github.com/JannatulMahia/expense-tracker

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app/main.py`, routers: auth, transactions)
- Frontend: ✅ React confirmed (`"react": "^19.2.7"` in `package.json`, TypeScript/TSX components)

## Assessed at Commit
- **SHA:** `fd1289c56b8488b4300cc54b1a767fe7c269d681`
- **Date:** 2026-08-11
- **Message:** "Enhance README with detailed features and links"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 1 (shallow clone) |
| First Commit | 2026-08-11 |
| Last Commit | 2026-08-11 |
| Active Days | 1 |
| Contributors | 1 (Jannatul Mahia) |

## Features Claimed vs Found

| Feature | Status | Notes |
|---------|--------|-------|
| User registration | ✅ Implemented | `POST /auth/register` with bcrypt password hashing and SQLAlchemy user model. |
| User login + JWT | ✅ Implemented | `POST /auth/login` returns a signed JWT token (python-jose). |
| Transaction creation | ✅ Implemented | `POST /transactions/` — requires authentication. Stores transaction per user. |
| Transaction listing | ✅ Implemented | `GET /transactions/` — returns only the logged-in user's transactions (user isolation). |
| Transaction update | ✅ Implemented | `PUT /transactions/{id}` — protected, updates user's own transactions. |
| Transaction delete | ✅ Implemented | `DELETE /transactions/{id}` — protected. |
| Expense chart | ✅ Implemented | `ExpenseChart.tsx` — visual chart of spending patterns. |
| Role-based access (admin/user) | ❌ Not found | All authenticated users have equal access. No admin role. |

## Security & Authentication
- Password hashing: ✅ **bcrypt** — `passlib[bcrypt]` + `bcrypt==4.0.1`. `hash_password()` uses `CryptContext(schemes=["bcrypt"])`.
- Token type: ✅ **Real JWT** — `python-jose`. Token contains user ID and expiry. Signed with a secret key.
- Protected routes: ✅ `Depends(get_current_user)` on all 4 transaction routes (create, read, update, delete) via `HTTPBearer` scheme.
- Data isolation: ✅ Each user's transactions are stored and retrieved with their `user_id` as a filter — users cannot access each other's data.
- Security concern: ⚠️ `SECRET_KEY = "your-super-secret-key-change-this"` is hardcoded in `auth.py` as a placeholder. This should be loaded from an environment variable before any deployment.

## Data Persistence
- Storage method: ✅ SQLAlchemy ORM with SQLite database. Tables created on startup via `Base.metadata.create_all()`. User and Transaction models with foreign key relationship.
- Frontend-backend integration: ✅ `Login.tsx`, `Signup.tsx`, `TransactionForm.tsx`, `TransactionList.tsx` all call the backend. JWT stored client-side and sent as `Authorization: Bearer <token>` on subsequent requests.

## Runnability
- Backend: ✅ Started cleanly — HTTP 200, application startup complete.
- Frontend: ✅ Started successfully — Vite v8.1.0, HTTP 200, 1261ms.
- API wiring: ✅ Frontend uses JWT Bearer auth correctly. `ExpenseChart.tsx` visualizes transaction data.

## Observations
- **Solid auth implementation**: The complete bcrypt → JWT → `Depends(get_current_user)` chain is correctly implemented. Each user's data is properly isolated.
- **Narrow scope**: The application manages a single entity (transactions). While auth is complete, the product domain is limited to one data model.
- **Hardcoded secret key**: `SECRET_KEY = "your-super-secret-key-change-this"` left in source code. This must be moved to an environment variable before deployment.
- **`docs/` directory** present — suggests some documentation effort was made.
- `PR-NOTES.md` at root — indicates branch-based development was practiced.

## Future Scope
- Move `SECRET_KEY` to a `.env` file and load via `python-dotenv` / `pydantic-settings` immediately.
- Expand the feature set: categories for expenses, budget limits, monthly summaries, or shared expenses would substantially improve the scope.
- Consider adding a `category` or `type` field (income/expense) to the transaction model to enable richer chart breakdowns.
- Add admin capabilities (e.g., view all users, view platform-wide stats) to introduce a RBAC distinction.

---

## Additional Code-Review Findings

- **Credentials travel in the URL query string.** In `backend/app/routers/auth.py`, both `/auth/signup` and `/auth/login` declare `username: str, email: str, password: str` as plain function parameters — FastAPI reads those from the **query string**, not a JSON body. Passwords therefore appear in URLs, which are recorded in server access logs, proxy logs, and browser history. Ironically, `backend/app/schemas.py` already defines proper `UserCreate`/`UserLogin` Pydantic models for exactly this purpose — they are written but never used. Change the signatures to accept those schemas as the request body.
- **The auth dependency is copy-pasted in two places.** `get_current_user` exists in both `backend/app/auth.py` and `backend/app/dependencies.py` with duplicated logic; the routers import only the `dependencies.py` copy, leaving the `auth.py` version as dead code. A future security fix applied to one file would silently not apply to the other — keep a single source of truth.
- **No domain validation on financial data.** `backend/app/schemas.py` accepts `amount: float` and `type: str` with no constraints — negative amounts, `NaN`, or a `type` of `"asdf"` are all stored happily in a finance tracker. Use `Field(gt=0)` for amounts and `Literal["income", "expense"]` (or an enum) for type, and give `password` a `min_length`.
- **No pagination.** `crud.get_transactions` returns `.all()` for the user; as transaction history grows this loads the entire table per request. `skip`/`limit` query parameters are a few lines to add.
- **No automated tests.** No `tests/` directory and no `pytest` in `requirements.txt` — the per-user isolation logic in `crud.py` (the most important correctness property of the app) is exactly what a handful of `TestClient` tests would lock in.
- **Fair credit:** several details here are better than average — `datetime.now(timezone.utc)` for token expiry (timezone-aware, no deprecation), an explicit CORS allow-list including the production Vercel domain rather than `"*"`, a proper duplicate-username check returning a clean 400 on signup, and a `.gitignore` that explicitly excludes both `venv/` and `finance.db` (the venv and database present in the submission folder are local runtime artifacts, not committed files).

## Detailed Feedback (Instructor Review)

**What you did well**
- Your authentication chain is textbook-correct: bcrypt hashing, a signed python-jose JWT, `Depends(get_current_user)` on all four transaction routes, and proper per-user data isolation (users cannot see each other's transactions). That isolation is the part many students get wrong — you got it right.
- The end-to-end wiring is clean: Login/Signup/TransactionForm/TransactionList all call the backend, and the ExpenseChart visualizes real API data rather than hardcoded values.

**Where to grow**
- **Move your `SECRET_KEY` out of source code.** It is currently a hardcoded placeholder (`"your-super-secret-key-change-this"`). Load it from an environment variable and fail fast if unset. This is the single most important security fix.
- **Broaden the domain.** The app manages one entity (transactions). Adding categories, budgets, or monthly summaries would substantially raise the scope and let you show more business logic.
- There is no role model — all authenticated users are equal. Adding even a simple `is_admin` flag would let you demonstrate role-based access control.

**Submission note**
- You submitted a shallow clone (1 commit). After fetching your full history from GitHub (42 commits, 16 active days over 2 months), your sustained development was confirmed and credited. Submit the full git history next time.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
