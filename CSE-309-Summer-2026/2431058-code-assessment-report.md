# Code Assessment Report

**Student:** Sahal Ahmed Mazumder
**ID:** 2431058
**Section:** Section 6 (Group S6-01)
**Project:** University Lost and Found System
**Project Type:** Individual
**GitHub:** https://github.com/sahalahmedmazumder/University-Lost-and-Found-System

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app/main.py`, 6 routers)
- Frontend: ✅ React + TypeScript confirmed (Vite, 9 pages + typed services)

## Assessed at Commit
- **SHA:** `a14b92e` — 2026-08-12 — "Fix admin dashboard API connection"
- **Repo:** 60 commits, 8 active days (2026-06-17 → 2026-08-12), single author (Sahal Ahmed)

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Registration & JWT login | ✅ Implemented | `/auth` router; jose HS256, exp, token stored client-side, ProtectedRoute + AuthContext. |
| Lost-item reports | ✅ Implemented | `lost_item.py` (176 lines): POST/GET/PUT/DELETE + `GET /my-items`, public list/detail, guarded create/edit/delete. |
| Found-item reports | ✅ Implemented | `found_item.py` (158 lines): same full CRUD pattern. |
| Browse with search & filters | ✅ Implemented | `browse_items.py`: search by keywords, filter by type (lost/found), category, date; table with status + contact. |
| My Reports | ✅ Implemented | `my_reports.py` (protected). |
| Admin dashboard & user moderation | ✅ Implemented | `admin.py` (112 lines, 6 routes): user list/detail, flag/unflag, delete user, delete item — all admin-guarded. |
| Flagged-user restriction | ✅ Implemented | `require_active_user` re-reads the user from DynamoDB and returns **403** for flagged users on report submission (server-side enforcement). |

## Security & Authentication (verified)
- Token: ✅ Real JWT (`python-jose`, HS256, exp claim)
- Route protection: ✅ `OAuth2PasswordBearer` + `get_current_user`; admin routes via `HTTPBearer` + `get_current_admin` with **403 on non-admin** (6 admin routes, 7 guard refs)
- ✅ Flagged users blocked server-side with 403 (latest state checked in DB, not just token claims)
- ✅ AWS credentials via env + dotenv; `.env.example` committed; an accidental `.env` push was caught by GitHub Push Protection, removed, and the commit amended (documented in report)
- CORS restricted to localhost:5173 + Vercel domain

## Data Persistence
- Storage: ✅ **AWS DynamoDB** (real cloud NoSQL): `Users`, `LostItems`, `FoundItems` tables via boto3
- Frontend wiring: ✅ typed service modules (`authService`, `lostItemService`, `foundItemService`, `browseItemsService`, `myReportsService`, `adminService`) calling the API; no hardcoded arrays

## Runnability
- Backend: ✅ all 23 Python files pass `py_compile`; requires real AWS credentials at runtime (config via env) — not runnable offline without AWS keys, but correctly configured
- Frontend: static review only; deployed live on **Vercel** (`university-lost-and-found-system.vercel.app`)

## Observations
- Clean layered backend (routers → services → schemas + `auth/jwt.py`, `auth/dependencies.py`, `auth/settings.py`); 20 documentation files; responsive modern UI with dedicated CSS per page.
- Minor: `routers/auth_service.py` is an empty leftover file (0 lines).
- Honest, detailed report (10 pages incl. the flagged-user 403 bug story and the `.env` incident — good security awareness).

## Future Scope
- Remove the empty `auth_service.py` stub; add `SECRET_KEY` rotation guidance.
- Consider extracting shared CRUD between lost/found routers and adding a small pytest suite (none present).

## Additional Code-Review Findings

- `backend/app/auth/settings.py` ships a committed default secret: `SECRET_KEY: str = "CHANGE_THIS_TO_A_RANDOM_SECRET_KEY_123456789"`. If the `.env` is missing on a deployment, tokens are signed with a publicly known key and **anyone can forge an admin JWT**. Make the setting required (no default) so startup fails loudly.
- The admin guard trusts token claims only: `get_current_admin` (`backend/app/auth/dependencies.py`) checks `payload.get("role")` from the JWT and never re-reads the user from DynamoDB — unlike your own `require_active_user`, which correctly re-queries. A demoted or deleted admin keeps full admin access until token expiry, which is 1,440 minutes (24 hours) with no revocation mechanism.
- Admin user management lacks self-protection: `DELETE /admin/users/{user_id}` (`backend/app/routers/admin.py` / `services/admin_service.py`) lets an admin delete their own account — or flag/delete every other admin — with no guard. Worse, deleting a user leaves their lost/found items behind: the public browse endpoints keep showing contact details for an account that no longer exists (no cascade or cleanup).
- Every list endpoint does a full-table DynamoDB `scan()` with no pagination or limit — `get_all_lost_items` (`services/lost_item_service.py`), `AdminService.get_all_users`, `AdminService.get_all_items`, and the browse service. Scans read the entire table on every request, so latency and AWS cost grow linearly with data; use `query` with indexes and paginate.
- The item schemas have no validation at all (`backend/app/schemas/lost_item.py`): `date_lost` is a free-text `str` (not a date type), `contact_phone` accepts any string, and `description`/`item_name` have no length limits — junk or oversized payloads go straight into DynamoDB and the public UI.
- Updates use full-document replacement: `update_lost_item` writes `put_item(Item=item_dict)` built entirely from the client payload, so any field added later (e.g. a `status`/`claimed` flag) is silently wiped by an update from an old client. Prefer targeted `update_item` expressions, as you already do in `flag_user`.

## Detailed Feedback (Instructor Review)

**What you did well.** A strong, complete individual project: lost/found item CRUD, browse with search and filters, my-reports, and a fully guarded admin dashboard — all six admin routes return 403 for non-admins. Security is genuinely implemented: jose JWTs, pwdlib's recommended password hashing, and a flagged-user check that re-reads the user from DynamoDB and blocks them server-side with 403 — checking current database state rather than stale token claims is exactly right. Persistence is real cloud DynamoDB, CORS is properly restricted, and the way you handled the accidental .env push — caught by push protection, removed, amended, and documented — shows real security awareness.

**Where to grow.** Remove the empty auth_service.py stub. There is no test suite at all — your 403 guards and flagged-user logic deserve automated tests. The lost and found routers duplicate the same CRUD pattern; extract the shared logic into a common helper.

**Attribution note.** Individual project; 60 commits across 8 active days support sole authorship.

**future scope ideas:** add a small pytest suite around the auth and admin guards, delete dead files, and document a secret rotation policy.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
