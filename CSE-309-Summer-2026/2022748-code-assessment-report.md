# Code Assessment Report

**Student:** Sheikh Jannatul Firdaus Nirjhor
**ID:** 2022748
**Section:** Section 5
**Project:** Afsheen By Sheikh — Fashion E-commerce Platform
**Project Type:** Individual
**GitHub:** https://github.com/sheikhnirjhor/Afsheen-By-Sheikh

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/main.py`, 10+ routes registered)
- Frontend: ✅ React confirmed (`"react": "^19.0.0"` in `package.json`, JSX components with hooks)

## Assessed at Commit
- **SHA:** `462558f48524903c45978efcf4eb7961c72bed73`
- **Date:** 2026-08-11
- **Message:** "frontend connected with backend"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 23 |
| First Commit | 2026-07-28 |
| Last Commit | 2026-08-11 |
| Active Days | 5 |
| Contributors | 1 (Sheikh Nirjhor) |

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Product browsing / shop | ⚠️ Partial | `ShopPage.jsx` uses `INITIAL_PRODUCTS` from local `src/data/products.js` (hardcoded list), not the backend `GET /api/products`. Backend endpoint exists and returns Firestore data. |
| Product detail page | ✅ Implemented | `ProductDetailPage.jsx` present with full display, gallery, reviews section. |
| Shopping cart | ✅ Implemented | `CartPage.jsx` + context state management. Cart persists in session. |
| Order placement | ✅ Implemented | `POST /api/orders` backend endpoint writes to Firestore. Frontend cart flows to order submission calling backend. |
| Product reviews | ✅ Implemented | `GET/POST /api/reviews/{product_id}` backend routes. Review section in product detail page. |
| User registration & login | ⚠️ Partial | `/api/auth/register` and `/api/auth/login` backend routes exist. Login compares passwords as plaintext strings. Frontend also has hardcoded demo credentials. |
| Admin dashboard | ✅ Implemented | `AdminDashboard.jsx` with product management, order management UI. Role `admin` checked in frontend. |
| Moderator dashboard | ✅ Implemented | `ModeratorDashboard.jsx` — separate role-based view. |
| Customer dashboard | ✅ Implemented | `CustomerDashboard.jsx` — order history, profile view. |
| Contact form | ✅ Implemented | `ContactPage.jsx` + `POST /api/contact` backend route, writes to Firestore. |

## Security & Authentication
- Password hashing: ❌ Plaintext — login route compares `user.get("password") == payload.password` directly. Passwords stored as-is in Firestore.
- Token type: ❌ No JWT or token system. Login returns the raw user object; no signed token issued.
- Protected routes: ❌ No `Depends()` on any route — all backend endpoints are publicly accessible without any credential.
- Role enforcement: ⚠️ Frontend-only — roles (`admin`, `moderator`, `customer`) checked in React components and context, but backend never validates who is calling which endpoint. Any unauthenticated user can call `POST /api/orders` or `DELETE /api/products/{id}`.
- Hardcoded credentials: ❌ `LoginPage.jsx` contains hardcoded demo accounts with plaintext passwords in JS source code.

## Data Persistence
- Storage method: ✅ Firebase Firestore (real cloud NoSQL database). Backend reads/writes Firestore via `firebase-admin` SDK. All entities (products, orders, reviews, users, contacts) persisted to Firestore.
- Frontend-backend integration: ⚠️ Partial — the frontend directly connects to Firestore via the Firebase JS SDK (`src/config/firebase.js`), partially bypassing the FastAPI backend. The `ShopPage` uses hardcoded local product data. Orders and contact forms do call the backend API.

## Runnability
- Backend: ⚠️ Not tested locally — requires Firebase service account credentials (`firebase-admin` SDK needs a credentials file). Would start if credentials were provided; code structure is valid.
- Frontend: ✅ Started successfully — Vite v6.4.3, HTTP 200, 4241ms
- API wiring: ⚠️ Partial — some features (orders, contact) call backend. Main product listing uses hardcoded local data. Firebase SDK also accessed directly from frontend.

## Observations
- **Broad feature scope**: 10 distinct features/pages implemented across admin, moderator and customer roles — the ambition and UI completeness is commendable.
- **Dual-path architecture risk**: Connecting the frontend directly to Firestore while also having a FastAPI backend creates a confusing architecture. Ideally, all data should flow through the API layer.
- **Plaintext passwords are a serious security flaw**: If this were a real application, all user credentials would be immediately exposed in the event of a database breach.
- **No backend route protection**: All endpoints are open — any caller can create, update or delete products without logging in.
- The last commit message "frontend connected with backend" shows the integration was done very close to submission deadline.

## Future Scope
- Implement `passlib[bcrypt]` for password hashing — never store or compare passwords as plain strings.
- Add JWT-based authentication: issue a signed token on login, verify it on every protected route using `Depends(get_current_user)`.
- Remove hardcoded credentials from frontend JavaScript source.
- Consolidate the data layer: have the frontend call only the FastAPI backend, and let the backend handle all Firestore access — remove the direct Firebase SDK from the frontend.
- Add `Depends(require_admin)` to admin-only routes like `DELETE /api/products/{id}` and `PUT /api/orders/{id}/status`.

## Additional Code-Review Findings

- **CORS misconfiguration.** `backend/main.py` sets `allow_origins=["http://localhost:5173", "http://localhost:3000", "*"]` together with `allow_credentials=True`. Browsers reject the wildcard-plus-credentials combination, and including `"*"` alongside explicit origins shows the security implications were not thought through — pick explicit origins only.
- **Order totals are client-trusted.** `OrderModel` accepts `total: float` from the request body, and `POST /api/orders` stores `"total": order.total` verbatim (line ~220) without recomputing from product prices in the database; `CartItem.price` is also client-supplied. A crafted request can check out for any amount.
- **Concrete data exposure (IDOR).** `GET /api/orders?email=...` (line ~231) returns any customer's full order history to anyone who knows or guesses their email address — no authentication and no ownership check. This is a privacy leak beyond the general "no route protection" issue.
- **Worth praising:** `backend/config.py` implements a complete in-memory fallback database that mirrors the Firestore API surface when no credentials file is present. That is a thoughtful zero-setup developer experience — and it means the backend actually can run locally without Firebase, contrary to initial appearances.
- **Good repo hygiene verified:** no `.env`, no `serviceAccountKey.json`, and no virtualenv are tracked in git — only `frontend/.env.example`. Given the security issues elsewhere, the credential-file hygiene here is genuinely clean.
- **Zero automated tests** exist anywhere in the repository — nothing guards the order/review logic against regressions.

## Detailed Feedback (Instructor Review)

**What you did well:** The scope here is genuinely broad — ten features across customer, moderator, and admin dashboards, with orders and reviews actually flowing through the backend to Firestore. Your 23 commits over 5 days show real effort, and the UI completeness is commendable.

**Where to grow:** The security posture of this project is poor, and I won't soften that. Passwords are stored and compared in plaintext. There is no JWT, no token system, and not a single `Depends()` guard on any backend route — anyone on the internet can call `DELETE /api/products/{id}` without logging in. Your role checks live only in React components, which protects nothing. You also left hardcoded demo credentials in the frontend source. Architecturally, connecting the frontend directly to Firestore via the Firebase SDK while also running a FastAPI backend defeats the purpose of having an API layer, and your shop page still renders hardcoded `INITIAL_PRODUCTS` instead of calling the backend. The final commit message — "frontend connected with backend" — confirms integration was left until the deadline.

**future scope ideas:** Hash passwords with bcrypt, issue signed JWTs on login, protect every write route with `Depends`, and route all data access through the backend. Fix the security layer before adding any new features.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
