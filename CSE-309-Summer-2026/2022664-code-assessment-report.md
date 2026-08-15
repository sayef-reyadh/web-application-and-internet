# Code Assessment Report

**Student:** Sudipta Saha
**ID:** 2022664
**Section:** Section 5
**Project:** Perfumology — Online Perfume Shop
**Project Type:** Individual
**GitHub:** https://github.com/iamsudiptasahadip/onlinePerfumeShop

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `app/main.py`, 3 routers: products, orders, contact)
- Frontend: ✅ React confirmed (`"react": "^18.2.0"` in `package.json`, TSX components)

## Assessed at Commit
- **SHA:** `debd1aaa5938b11482ca10686f455e055c98f688`
- **Date:** 2026-08-13
- **Message:** "Merged remote changes"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 3 |
| First Commit | 2026-06-13 |
| Last Commit | 2026-08-13 |
| Active Days | 2 |
| Contributors | 1 (Sudipta Saha) |

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Product browsing / shop | ⚠️ Partial | Frontend `Shop.tsx` imports from a static local file (`src/data/product.ts`) — a single hardcoded product. Does not call the backend API. |
| Product detail page | ⚠️ Partial | `ProductDetailPage.tsx` exists — but displays the same hardcoded product, not fetched from backend. |
| Shopping cart | ⚠️ Partial | `Cart.tsx` + `cartStore.ts` manage cart state locally via Zustand. No API call to persist cart or submit order to backend. |
| Order placement | ⚠️ Partial | Backend has a full `POST /api/orders` endpoint with PostgreSQL models. However, the frontend cart does not call this endpoint — no API integration found. |
| Contact form | ⚠️ Partial | `Contact.tsx` page exists. Backend has `POST /api/contact`. No API call found in frontend contact page. |
| User authentication / login | ❌ Not found | No login page, no signup, no user model, no auth routes in backend or frontend. |
| Admin panel | ❌ Not found | No admin routes or frontend panel found. |

## Security & Authentication
- Password hashing: ❌ Not implemented — no user model exists
- Token type: ❌ No authentication system at all
- Protected routes: ❌ None — all routes are open
- Role enforcement: ❌ No roles exist

## Data Persistence
- Storage method: ⚠️ Backend uses PostgreSQL (SQLAlchemy ORM, proper models for Product, Order, OrderItem). However, the frontend does not call the backend — product data is served from a hardcoded local TypeScript file.
- Frontend-backend integration: ❌ Disconnected — no API calls found in frontend source. The backend API exists but is never used by the frontend.

## Runnability
- Backend: ❌ Failed to start — pydantic `ValidationError` on startup: `DATABASE_URL` field required but not loaded correctly. The `.env` file contains an unexpected `gh_pat` key which conflicts with the pydantic Settings model, causing an `extra_forbidden` validation error.
- Frontend: ✅ Started successfully — Vite v5.4.21, HTTP 200, 1063ms
- API wiring: ❌ Frontend operates fully standalone — no fetch/axios calls to backend found

## Observations
- The backend is well-structured: proper SQLAlchemy models, pydantic schemas, FastAPI routers with clean separation. The code quality of the backend is good.
- However, the frontend is entirely decoupled — it renders one hardcoded product from a local data file and manages state locally only.
- The `.env` file contains a raw GitHub PAT (`gh_pat_...`) that was accidentally committed — this is a security concern as the token may grant repository access.
- Only 3 commits across 2 days suggests the project was assembled very quickly before submission.

## Future Scope
- Connect the frontend to the backend: replace `import { product } from '../data/product.ts'` with a `fetch('/api/products')` call in `Shop.tsx`.
- Add user authentication: implement a `/api/auth` router with login/signup, JWT tokens, and bcrypt password hashing.
- Remove the GitHub PAT from `.env` and rotate the token immediately — committed credentials should be treated as compromised.
- Remove the `.env` file from version control and add it to `.gitignore`.
- Increase commit frequency throughout development to better reflect the development journey.

## Additional Code-Review Findings

- **Price-tampering vulnerability (serious).** `POST /api/orders` trusts client-supplied prices: `OrderItemBase` in `app/schemas.py` accepts `unit_price` and `total_price` from the request body, and `create_order` in `app/routes/orders.py` computes `item_total = item.unit_price * item.quantity` without ever reading `product.price` from the database. A crafted request could order any perfume for a fraction of its price. Always derive prices server-side from the Product row.
- **Repo hygiene, verified via `git ls-files`:** `backend/venv/` is committed with roughly 2,900 tracked files, along with 9 `__pycache__` bytecode files under `backend/app/` and `.vscode/settings.json`. None of these belong in version control — remove them with `git rm -r --cached` and extend `.gitignore`.
- **Insecure defaults in `app/config.py`:** `SECRET_KEY = "change-me-in-production"` and `DEBUG = True` are the defaults, so the app runs with a publicly known secret and debug mode enabled unless explicitly overridden.
- **Startup fragility:** `Base.metadata.create_all(bind=engine)` runs at import time in `app/main.py`, so the whole application crashes if the database is unreachable — one reason the backend failed to boot during assessment. Move schema creation to an explicit migration step (e.g., Alembic) and let the app start independently.
- **Worth acknowledging:** `create_order` does perform real server-side validation — product-not-found 404s, insufficient-stock 400s, and `EmailStr` email validation in the schema. The backend logic is sound; the tragedy is that the frontend never calls it.
- **No automated tests** exist anywhere in the repository — nothing guards the order/stock logic against regressions.

## Detailed Feedback (Instructor Review)

**What you did well:** Your backend code is well-structured — proper SQLAlchemy models, pydantic schemas, and cleanly separated FastAPI routers for products, orders, and contact. The quality of that code is not in question.

**Where to grow:** I have to be direct here. Your frontend makes zero API calls. The shop renders a single hardcoded product from a local TypeScript file, the cart never reaches your `POST /api/orders` endpoint, and the contact form never calls the backend. As submitted, the two halves of this project do not talk to each other at all — the application does not function as a full-stack system. There is also no authentication of any kind: no login, no user model, no protected routes. Worse, you committed a raw GitHub personal access token in your `.env` file — that token is compromised and must be rotated immediately. Three commits across two active days makes clear this was assembled at the last moment, and the gap between what a full-stack project requires and what exists here is significant.

**future scope ideas:** Rotate the leaked token today. Then wire the frontend to the API, add real authentication, and commit your work regularly as you build.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
