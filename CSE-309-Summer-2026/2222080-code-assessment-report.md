# Code Assessment Report

**Student:** Alif Al Fattah
**ID:** 2222080
**Section:** Section 5 (Group S5-21)
**Project:** SubWise – Smart Subscription Analytics & Budget Management System
**Project Type:** Individual
**GitHub:** https://github.com/Alifsvoid/subwise-subscription-tracker

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/main.py`, 7 routes)
- Frontend: ✅ React confirmed via **Next.js** (TypeScript + Tailwind CSS)

## Assessed at Commit
- **SHA:** `7c037b6` — 2026-08-08 — "Update frontend currency options and formatting"
- **Repo:** 48 commits, 10 active days (2026-06-17 → 2026-08-08), single author (Alifsvoid)

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| JWT registration & login | ✅ Implemented | `/register`, `/login`; real PyJWT token (7-day expiry), DB re-query of user on each request. |
| Subscription CRUD | ✅ Implemented | `/subscriptions` GET/POST/PUT/DELETE with full attribute set (category, price, billing cycle, next billing date, currency, payment method, trial, active). |
| User-data isolation | ✅ Implemented | Every query scoped by `user_id == current_user.id`. |
| Interactive dashboard analytics | ✅ Implemented | Normalized monthly spend, annualized projection, average service cost, top expense — client-side over the API data. |
| Multi-currency conversion (USD/BDT/EUR) | ✅ Implemented | `EXCHANGE_RATES` mapping + centralized `formatPrice` helper; dynamically recalculates totals and formats (latest commit). |
| Budget tracking with progress bars | ✅ Implemented | Monthly spending cap with visual consumption percentage. |
| CSV export | ✅ Implemented | Browser download from subscription state. |
| Renewal email notifications | ⚠️ Stub | `send_email_notification_task` only prints to stdout; BackgroundTasks queue is real, but no actual SMTP send. |

## Security & Authentication
- Token type: ✅ Real JWT (PyJWT, HS256, 7-day expiry)
- Route protection: ✅ all 5 subscription routes use `Depends(get_current_user)` via HTTPBearer (14 Depends total); ownership enforced in every query
- Password hashing: ⚠️ PBKDF2-SHA256 with 100k iterations, but a **hardcoded global salt** (`"subwise_secure_salt_2026"`) — no per-user salt, so identical passwords produce identical hashes and offline precomputation attacks are viable
- ⚠️ **Hardcoded SECRET_KEY in source**: `"subwise_jwt_secret_key_change_in_production"` — must come from environment
- No role model (single user type — acceptable for a personal tracker); CORS correctly restricted to localhost:3000

## Data Persistence
- Storage: ✅ SQLite + SQLAlchemy ORM (`subwise.db`), users + subscriptions tables with relationship and cascade delete
- Frontend wiring: ✅ Next.js client components fetch the API with the stored JWT; state managed via React hooks

## Runnability
- Backend: static review (no test suite present); code is small and coherent (317 lines), `main.py` imports cleanly (fastapi, sqlalchemy, jwt, pydantic EmailStr)
- ⚠️ Duplicate `app = FastAPI(title="SubWise API")` statement (harmless but sloppy)

## Observations
- **Repo hygiene issue:** the full `backend/venv/` (≈75 MB of site-packages) is committed, plus `subwise.db`; also stray `frontend/page.tsx` and committed `frontend/AGENTS.md` / `frontend/CLAUDE.md` at the repo root of frontend.
- Good: scoped `git add` workflow documented (`.vs/` lockout solved via .gitignore), 22 documentation files (01–22 + feasibility analysis), Pydantic schemas, `exclude_unset` partial updates.
- The report (6 pages) is honest and specific about challenges (currency recalculation bug, schema migration, git lockouts).

## Future Scope
- Move `SECRET_KEY` to environment (fail fast if unset) and use a per-user random salt (e.g., `secrets.token_bytes(16)`) for password hashing.
- Implement real email delivery (SMTP/sendgrid) or drop the claim.
- Remove `backend/venv/`, `subwise.db`, and stray root files; add a proper `.gitignore`.

---

## Additional Code-Review Findings

- **Soft-deleted subscriptions reappear in the list.** `DELETE /subscriptions/{id}?hard_delete=false` sets `is_active = False`, but `GET /subscriptions` in `backend/main.py` filters only by `user_id` — never by `is_active` — so every "archived" subscription still shows up in the user's list and dashboard totals. Either filter `is_active == True` on read or drop the soft-delete option.
- **Auth error responses leak internals and use wrong status codes.** `get_current_user` returns `detail=f"Invalid or expired token: {str(e)}"` — raw JWT exception text goes to the client, and the broad `except Exception` masks programming errors as auth failures. Additionally, `/login` raises **400** for wrong credentials; 400 means "malformed request" — authentication failures should be 401 (and identical for "no such email" vs "wrong password").
- **User enumeration via registration.** `POST /register` returns `"Email is already registered"`, which lets anyone probe which email addresses hold accounts. Return a generic message (or accept-and-notify flow) instead.
- **Dates and enums stored as free-form strings.** `next_billing_date` is a plain `str` in both the schema and the DB column, and `billing_cycle` is an unconstrained `str` (the "monthly or yearly" contract exists only as a code comment). `price: float` accepts negative values. Use `datetime.date`, `Literal["monthly", "yearly"]`, and `Field(gt=0)` so bad data is rejected at the edge instead of corrupting analytics.
- **No password policy.** `UserAuth.password` has no `min_length` — a one-character password hashes and stores fine.
- **Fair credit:** several details are above average — `get_current_user` re-queries the database on every request (revoked/deleted users lose access immediately), registration uses `EmailStr` validation and checks for duplicate emails before insert, token timestamps are timezone-aware (`datetime.now(timezone.utc)`), and the `SubscriptionUpdate` + `exclude_unset=True` pattern is the correct way to do partial updates.
- **Test coverage remains the missing piece** — the codebase is a single 317-line `main.py`, which makes it the easiest project in the batch to cover: a few `TestClient` cases for register/login/isolation would have caught the soft-delete bug above.

## Detailed Feedback (Instructor Review)

**What you did well**
- Real, enforced authentication: PyJWT (HS256, 7-day expiry) via HTTPBearer, `Depends(get_current_user)` on all 5 subscription routes, and strict per-user ownership scoping in every query. The full-stack wiring (Next.js client storing the JWT and calling the API) is correct.
- Impressive feature breadth for a solo project: subscription CRUD, dashboard analytics (monthly/annualized spend, top expense), multi-currency conversion, budget caps with progress bars, and CSV export — all wired end-to-end.
- Your 6-page report is honest and specific about real challenges (the currency recalculation bug, schema migration, git lockouts) — that kind of candor is exactly what good engineering write-ups look like.

**Where to grow**
- **Two credential-handling fixes:** (1) the `SECRET_KEY` is a hardcoded literal in source — move it to an environment variable; (2) your PBKDF2 uses a single hardcoded global salt, so identical passwords produce identical hashes. Use a per-user random salt (`secrets.token_bytes(16)`).
- **Repo hygiene:** you committed a ~75 MB `backend/venv/` and the `subwise.db` database. Add a `.gitignore` and never commit dependencies or data files.
- The renewal-email feature is a print-stub — either implement real SMTP delivery or present it honestly as a placeholder.

**Submission note**
- The submitted repo was a shallow clone (1 local commit), but your full history was confirmed via the GitHub API: 48 commits across 10 active days (2026-06-17 → 2026-08-08). Your development process is genuine and well-spread. In future, submit the full `.git` history so this is immediately visible.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
