# Code Assessment Report

**Student:** Kazi Ismat Nahar Epthi
**ID:** 2330813
**Section:** Section 6
**Project:** Finance Tracker App (no project name submitted in student_map)
**Project Type:** Individual
**GitHub:** https://github.com/kaziepthii/Cse_309

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/main.py`)
- Frontend: ✅ React + TypeScript confirmed (`frontend/`, Vite)

## Assessed at Commit
- **SHA:** `ab8f103` — 2026-08-13 — "Final project: Added Wallet, Budget, Savings Goals, Future Bills, Dark Mode, Toast Notifications, Fixed Budget-Tracking Integration"
- **Repo:** 17 commits, 7 active days (2026-06-30 → 2026-08-13), single author (kaziepthii + "kazi ismat epthi" identities)

## Features Found (21 routes)
- Auth: register + login
- Transactions: CRUD + `/api/summary` (with budget auto-update)
- Wallet: balance, add/withdraw, wallet transaction history
- Future bills: create/list/pay with reminder flag
- Savings goals: create/list, add money to goal
- Monthly budget: create/list per category + **budget alert** endpoint
- Frontend: Wallet Summary/Budget/Transactions, Monthly Budget, Savings Goals, Future Bills, Budget Alerts, AddTransactionForm, search bar, dark mode, toast notifications — wired to backend via fetch (`App.tsx`, 786 lines)

## Security & Authentication — ⚠️ NONE
- ❌ **No password hashing** — passwords compared in **plaintext**: `SELECT * FROM users WHERE username = ? AND password = ?`
- ❌ **No tokens/sessions** — login returns `user_id` + username; no JWT, no `Depends`, no auth middleware anywhere (0 Depends in 807 lines)
- ❌ **No authorization** — every protected route takes `user_id` as a client-supplied argument → any user can read/delete another user's transactions, wallet, bills, budgets (IDOR)

## Data Persistence
- ✅ Real **SQLite** database (`finance.db` — committed, plus duplicate at repo root): users, transactions, wallet, wallet_transactions, future_bills, savings_goals, monthly_budget; init_db with FKs

## Runnability
- ✅ Single-file backend (807 lines) compiles; Postman collection committed (`.postman/resources.yaml`)
- Frontend runs on Vite dev server pointed at `localhost:8000`

## Observations
- Good feature scope and honest single-developer effort (17 commits, docs set complete: SRS, ERD, DFD, TDD, database design + 6 diagram images).
- The app is a demo-data-level implementation: functional, but zero security — the biggest possible deduction.
- `finance.db` committed twice (repo root + backend/), one giant `main.py`/`App.tsx` — maintenance debt.

## Future Scope
- Add bcrypt password hashing + JWT (PyJWT) with `Depends(get_current_user)`; derive the user from the token instead of the query string; remove committed `.db` files; split main.py/App.tsx.

## Additional Code-Review Findings

- The repository commits an entire virtual environment: roughly 2,400 files under `venv/` are git-tracked (including platform-specific artifacts like `venv/Include/site/python3.12/greenlet/greenlet.h` and `.pyc` caches). This bloats the repo enormously and makes it non-portable — `venv/` belongs in `.gitignore`.
- The budget-tracking integration has a correctness bug: every `spent_amount` update omits the category filter. In `add_transaction`, `update_transaction`, `delete_transaction`, and `pay_future_bill` (all in `backend/main.py`) the statement is `UPDATE monthly_budget SET spent_amount = spent_amount + ? WHERE user_id = ? AND month = ?` — with no `AND category = ?`. Since budgets are per-category, a single expense inflates the spent amount of **every** budget row for that month, so `/api/budget/alert` reports wrong numbers.
- Dates are handled inconsistently: wallet/bill flows stamp server-side `datetime.now()` (`"%Y-%m-%d"` / `"%m/%Y"`), while transaction months are derived by splitting the client-supplied free-text `date` string on `"/"` (`backend/main.py`, `add_transaction`). Two different month derivations mean budget rows silently fail to match whenever the client sends e.g. `2026-08-15` instead of `08/15/2026`, and no date validation exists anywhere.
- No numeric validation: every money model (`TransactionCreate`, `WalletTransactionCreate`, `FutureBillCreate`, `SavingsGoalCreate`) declares `amount: float` with no `gt=0`, so a negative "expense" actually **increases** the wallet balance. Likewise `type` is an unconstrained string — any value other than `"income"` is silently treated as an expense.
- Deleting or editing a transaction reverses the wallet balance and budget totals but never removes or updates the corresponding `wallet_transactions` row (`delete_transaction` / `update_transaction` in `backend/main.py`), so the wallet history permanently disagrees with the actual balance — orphaned entries accumulate.
- The "session" is just a React `useState` integer (`frontend/src/App.tsx`, `userId` state) — refreshing the page silently logs the user out, and every request embeds the raw `?user_id=` in the URL (e.g. `/api/wallet?user_id=3`), which also leaks account identifiers into server and proxy logs. `sqlalchemy` is additionally declared in `backend/requirements.txt` but never imported — raw `sqlite3` is used throughout.

## Detailed Feedback (Instructor Review)

**What you did well.** You delivered a genuinely broad finance tracker — transactions, wallet, future bills, savings goals, monthly budgets with alerts — backed by a real SQLite database and a working React frontend, plus a complete documentation set. The feature scope and honest solo effort across 17 commits are commendable.

**Where to grow.** I have to be blunt: this application has no security whatsoever, despite jose, passlib, and bcrypt appearing in requirements.txt — none of them are ever imported. Passwords are stored and compared in plaintext, login issues no token or session, and every "protected" route trusts a client-supplied `user_id`, so any user can read, modify, or delete any other user's data. Authentication is claimed but entirely absent — that is the most serious flaw a project like this can have. Committing finance.db twice and maintaining single 800-line files are habits you must also fix.

**Attribution note.** Individual project; all work is your own.

**future scope ideas:** hash passwords with passlib/bcrypt, issue JWTs, protect every route with a `get_current_user` dependency, derive the user from the token — never the query string — and remove committed database files.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
