# Code Assessment Report

**Student:** Rayed Mahmud
**ID:** 2120010
**Section:** Section 5
**Project:** Finvo — Cloud Kitchen Financial Dashboard
**Project Type:** Individual
**GitHub:** https://github.com/rayed15/Finvo

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/main.py`, 18 routes registered)
- Frontend: ✅ React confirmed (`"react": "^19.2.6"` in `package.json` at repo root, JSX components)

## Assessed at Commit
- **SHA:** `646429df8f67266906a873ad607fa5e95a631040`
- **Date:** 2026-08-13
- **Message:** "Made fina changes"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 12 |
| First Commit | 2026-06-18 |
| Last Commit | 2026-08-13 |
| Active Days | 3 |
| Contributors | 1 (Rayed Mahmud) |

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Delivery costs management | ✅ Implemented | `GET/POST/DELETE /api/delivery-costs` backend routes. `DeliveryCosts.jsx` frontend page. |
| Commissions management | ✅ Implemented | `GET/POST/DELETE /api/commissions` backend routes. `Commissions.jsx` frontend page. |
| Fixed costs management | ✅ Implemented | `GET/POST/DELETE /api/fixed-costs` backend routes. `FixedCosts.jsx` frontend page. |
| Ingredients management | ✅ Implemented | `GET/POST/DELETE /api/ingredients` backend routes. `Ingredients.jsx` frontend page. |
| Order sources management | ✅ Implemented | `GET/POST/DELETE /api/order-sources` backend routes. `OrderSources.jsx` frontend page. |
| Financial dashboard / summary | ✅ Implemented | `GET /api/dashboard/summary` backend route. `Dashboard.jsx` frontend page. |
| User authentication / login | ❌ Not found | No auth routes, no login page, no user model. All endpoints are open to any caller. |
| Role-based access | ❌ Not found | No roles, no RBAC, no admin/user distinction. |

## Security & Authentication
- Password hashing: ❌ Not implemented — no user system at all
- Token type: ❌ No authentication system whatsoever
- Protected routes: ❌ All 18 API endpoints are publicly accessible with no credentials
- Role enforcement: ❌ No roles exist

## Data Persistence
- Storage method: ✅ Firebase Firestore (primary). Backend also has a SQLite fallback implemented (`USE_SQLITE` env var). The code gracefully falls back to SQLite if Firebase credentials are not provided — thoughtful design.
- Frontend-backend integration: ✅ Fully wired — `src/api/client.js` handles all fetch calls to `API_BASE_URL`. `src/api/services.js` maps all backend endpoints. No hardcoded data.

## Runnability
- Backend: ⚠️ Process starts (`INFO: Started server process`) but startup hangs waiting for Firebase service account credentials. The SQLite fallback logic exists in code but requires `USE_SQLITE=true` environment variable. Code structure is valid — deployment-ready with Dockerfile and Procfile present.
- Frontend: ✅ Started successfully after `npm install` — Vite v8.0.16, HTTP 200, 6810ms.
- API wiring: ✅ Frontend is fully wired via `client.js` and `services.js`.

## Observations
- **Well-scoped tool**: Finvo is a focused financial dashboard for cloud kitchens. The 5 cost-management modules + dashboard form a coherent, useful product.
- **Thoughtful deployment setup**: Dockerfile and Procfile present — the student prepared this for cloud deployment (Heroku/Railway). This shows engineering maturity beyond the typical student project.
- **Firebase + SQLite fallback**: The dual-backend design (Firebase primary, SQLite fallback) is a smart architectural decision for a project with cloud database dependencies.
- **Completely missing authentication**: The entire platform is open — any user can read, add, or delete any financial data. This is the most significant gap.
- **Only 3 active commit days** despite starting in June — most of the work was done in short bursts.
- `node_modules` not committed — correct, but requires `npm install` before running.

## Future Scope
- Add user authentication: a simple `POST /api/auth/login` endpoint with JWT (python-jose) and bcrypt password hashing would significantly improve the security posture.
- Protect all write routes (`POST`, `DELETE`) with `Depends(get_current_user)` once auth is in place.
- Document the `USE_SQLITE=true` fallback in the README — this makes the project much easier to run locally without Firebase credentials.
- Increase development regularity — the project has good bones but shows signs of being developed in a single intensive session.

## Additional Code-Review Findings

- **The Firestore path is dead code.** `USE_FIRESTORE = False` is assigned once at module level in `backend/main.py` (line ~181) and is never set to `True` anywhere — it is only read. Even with valid Firebase credentials configured, every request is served by the SQLite fallback. Either wire the flag to successful Firebase initialization or be honest in the README that SQLite is the actual datastore.
- **CORS is wide open:** `allow_origins=["*"]` (line ~84) allows any website origin to call the API from a visitor's browser — compounding the missing authentication.
- **Deprecated lifecycle, brittle startup:** `@app.on_event("startup")` is deprecated in current FastAPI (use a `lifespan` handler), and because the handler re-raises on Firebase failure, the app hard-crashes at boot instead of degrading to SQLite — which defeats the purpose of having a fallback.
- **Monolithic backend:** a single ~450-line `backend/main.py` holds Firebase initialization, the SQLite schema DDL, all five pydantic models, and all 18 routes. Splitting into `routers/`, `models/`, and `db/` modules is the natural next refactor.
- **Good SQL hygiene worth praising:** every SQLite query is parameterized (`conn.execute("INSERT INTO delivery_costs VALUES (?,?,?,?,?,?)", ...)`), and `ensure_db()` returns a clean HTTP 503 instead of an unhandled 500 when the datastore is unavailable.
- **Zero automated tests** exist anywhere in the repository — nothing protects the cost-calculation endpoints from regressions. On the positive side, Firebase credentials are accepted only via environment variables and documented in `backend/.env.example`; no service-account JSON is committed.

## Detailed Feedback (Instructor Review)

**What you did well.** Finvo is a coherent, well-scoped product: five cost-management modules plus a dashboard summary, all wired end-to-end through a clean `client.js`/`services.js` layer with no hardcoded data. The Firestore-primary/SQLite-fallback design is a genuinely smart architectural choice, and including a Dockerfile and Procfile shows deployment awareness most peers lack.

**Where to grow.** Bluntly: this application has no security at all. There is no login, no user model, no token, and no guard — every one of your 18 endpoints, including all deletes, is open to anyone on the internet. For a financial dashboard, that is disqualifying in any real setting, and it was a core course requirement. Your commit pattern is also a concern: 12 commits compressed into 3 active days across two months reads as last-minute bursts, not sustained engineering.

**Submission note.** No PDF report was submitted with this project, and the git history, while present, is too thin to demonstrate a steady development process.

**future scope ideas:** add a JWT login (python-jose + bcrypt), guard every write route with `Depends(get_current_user)`, document `USE_SQLITE=true` in the README, and commit incrementally.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
