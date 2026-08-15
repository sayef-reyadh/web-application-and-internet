# Code Assessment Report

**Student:** Md. Touhidul Hasan
**ID:** 1820450
**Section:** Section 6
**Project:** PayPulse — Subscription Tracker
**Project Type:** Individual
**GitHub:** https://github.com/iamiash/pay-pulse

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/`, api/v1 endpoints + core + services + db)
- Frontend: ✅ React + TypeScript confirmed (`frontend/`, Vite + Tailwind, 30+ pages)

## Assessed at Commit
- **SHA:** `e910dca` — 2026-08-13 — "Updated complete project"
- **Repo:** **1 commit**, 1 active day (2026-08-13), single author — entire project pushed in one shot

## Features Found — what actually runs
Mounted routers (`app/api/v1/router.py`): auth, users, subscriptions, wallet, notifications — **34 live routes**:
- Auth (6): register, login, email verify, resend user id, forgot/reset password
- Users (8): profile get/update, avatar upload/delete, verify credentials, change password, delete account
- Subscriptions (5): list, create, update, search, delete
- Wallet (12): bank accounts + credit cards + mobile banking — CRUD each, payment data **encrypted at rest** (Fernet)
- Notifications (3): list, mark read, read-all

Frontend: Dashboard, Subscriptions, Renewals, Analytics, SmartSavings, WalletView, AccountSettings, Profile, Login/Register, VerifyEmail, ForgotPassword, Landing — wired through typed API modules.

## ⚠️ Dead code found (significant)
- `endpoints/admin.py` (10 routes: dashboard metrics, users, subscriptions, wallet, audit-logs, security) and `endpoints/reports.py` (**0 routes**, empty) are **never mounted** in `router.py`/`main.py`
- The frontend's 10 Admin pages + Reports page call `/admin/*` and `/reports/*` via `adminApi.ts`/`reportApi.ts` — these would all return **404** in the running app
- `pdf_service.py` exists but is unreachable (no route calls it)

## Security & Authentication (verified)
- ✅ bcrypt password hashing
- ✅ Real JWT (PyJWT, HS256, exp), OAuth2PasswordBearer
- ✅ `get_current_user` re-queries DB + account-status guards (BANNED/SUSPENDED/DEACTIVATED → **403**)
- ✅ `require_admin` (ADMIN/SUPER_ADMIN → 403) defined for admin endpoints
- ✅ **Fernet AES encryption** of stored payment credentials
- ✅ Email verification + password reset flows
- ⚠️ Hardcoded `SECRET_KEY`/`FERNET_KEY` in `config.py` (no env override); SMTP creds empty (email features non-functional in practice)

## Data Persistence
- ✅ SQLAlchemy ORM + SQLite (`paypulse.db` committed), User + payment tables, init_db seed

## Runnability
- ✅ 22/22 backend files pass `py_compile`
- Frontend builds (static review only); runtime issues: admin/reports 404, email features need real SMTP

## Observations
- Excellent engineering instincts (encryption, audit trail, account-status guards, typed services) — but this is a **single-commit repo** (no incremental history, no issues/PRs, no tests, no CI), and a whole claimed feature tier (Admin + Reports) is dead code.
- 23 docs present (SRS/TDD/ERD/DFD etc.).

## Future Scope
- Mount the admin/reports routers (2-line fix) or remove the dead UI; split work into incremental commits; add env-only secrets; add tests.

## Additional Code-Review Findings

- **The wallet encryption key is regenerated on every restart.** [security.py](repo/backend/app/core/security.py) reads `getattr(settings, "ENCRYPTION_KEY", None)`, but [config.py](repo/backend/app/core/config.py) only defines `FERNET_KEY` — the lookup always fails, so `Fernet.generate_key()` runs at import time and every process restart produces a **new random key**. All previously encrypted wallet credentials become permanently undecryptable, and `decrypt_sensitive_data` then swallows the exception and returns raw ciphertext to API clients as if it were data.
- **Two contradictory token lifetimes.** `config.py` sets `ACCESS_TOKEN_EXPIRE_MINUTES` to 7 days, while `create_access_token()` in `security.py` independently defaults to `timedelta(minutes=43200)` — 30 days. Neither matches the other, and there is no refresh or revocation mechanism for these long-lived tokens.
- **A GET endpoint mutates the database.** `list_subscriptions` in [subscriptions.py](repo/backend/app/api/v1/endpoints/subscriptions.py) calls `check_and_generate_renewal_notifications(...)` and `sync_subscription_status(...)` (which commits) — a read-only request with write side effects, which breaks caching, retries, and basic HTTP semantics.
- **Raw internal errors leak to clients.** `create_subscription` returns `detail=f"Failed to save subscription: {str(e)}"`, exposing raw SQLAlchemy exceptions, while several `except Exception: pass` blocks (`sync_subscription_status`, `calculate_tenure_months`) silently discard real failures.
- **Dates are handled as strings and naive datetimes.** `next_billing_date[:10]` is parsed with `strptime` across endpoints, and `create_access_token` uses the deprecated, timezone-naive `datetime.utcnow()` — fragile, timezone-blind date logic for a billing-adjacent app.

## Detailed Feedback (Instructor Review)

**What you did well.** PayPulse shows excellent engineering instincts: Fernet-encrypted payment credentials at rest, real PyJWT tokens with `OAuth2PasswordBearer`, account-status guards (BANNED/SUSPENDED/DEACTIVATED), a `require_admin` dependency, email verification and password-reset flows, and 34 working routes across auth, users, subscriptions, wallet, and notifications behind a polished 30-page React frontend.

**Where to grow.** Be honest with yourself about what actually runs: your entire admin module (10 routes) and reports module are dead code — never mounted — so the 11 frontend pages calling `/admin/*` and `/reports/*` would all 404 in production. That is a whole claimed feature tier that does not exist at runtime. Your `SECRET_KEY` and `FERNET_KEY` are hardcoded in `config.py`, SMTP is unconfigured so email features cannot work, and there are no tests. Shipping dead code as a feature is worse than not building it.

**Submission note.** The entire project arrived as a single commit on a single day. With no incremental history, issues, or PRs, I cannot verify your process or how this code evolved — that hurts your credibility even when the code is good.

**future scope ideas:** mount the admin/reports routers or delete the dead UI, move all secrets to environment variables, write tests, and never submit a one-commit repo again.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
