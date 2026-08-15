# Code Assessment Report

**Student:** Moumita Singh Roy Moon
**ID:** 2312186
**Section:** Section 6 (Group S6-07)
**Project:** BiteBuddy — NextGen Food Ordering Platform
**Project Type:** Team (with Farhana Ahmed 2312079, Jannatun Naim 2330097)
**GitHub:** https://github.com/moon496/BiteBuddy-NextGen-Food-Ordering-Platform

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/`, 8 routers)
- Frontend: ✅ React confirmed (`frontend/`, Vite)

## Assessed at Commit
- **SHA:** `71c37fe` — 2026-08-13 — "update"
- **Repo:** 91 commits, 20 active days (2026-06-18 → 2026-08-13)
- **This student:** 42 commits (`Moumita singh roy moon` 41 + `moon496` 1), **11 distinct days** (06-18 → 08-13)

## Individual Contribution — verified per-path authorship

| Claimed (report PDF) | Git record |
|----------------------|-----------|
| Menu display + cart frontend | ✅ `MenuItems.jsx` 8, `cartpage.jsx` 9, `cartApi.js` 5, `Checkout.jsx` 5, `OrderStatus.jsx` 5 |
| Backend cart/order/payment routes | ✅ `cart_routes.py` 3, `order_routes.py` 5, `payment_routes.py` 3, `address_routes.py` 2 |
| Admin order management (payment-type-aware status) | ✅ `admin_routes.py` 4, `AdminDashboard.jsx` 4 |
| Auth frontend integration (token lifting, bug fixes) | ✅ `utils/auth.js` 2, `authApi.js` 3, `Login.jsx` 3 |
| Deployment (Render backend, Vercel frontend, CORS regex) | ✅ `main.py` 15 commits — CORS `allow_origin_regex`, auto-seed on startup |
| Database models + seeds | ✅ `model.py` 5, `seed.py`, `seed_menu.py` |

Her report's claims are backed by git — she is the team's deployment owner and cart/order/payment lead.

## Features (team app, 8 routers, 40+ routes)
- Auth (register/login/me/logout/update/delete) • Cart (get/post/put/delete) • Orders (create, per-user list, status track, confirm payment) • Payment (initiate/callback) • Admin (menu CRUD, order status, revenue, admin mgmt, coupons, banned-users) • Coupons • Reviews • Address book
- Frontend: MenuItems, CartPage, Checkout, Payment, OrderStatus, Reviews, Coupon, AddressBook, AdminDashboard, Dashboard, NotificationModal (COD delivery notification)

## Security & Authentication (verified)
- ✅ bcrypt + real JWT (PyJWT, HS256, exp) in `auth_utils.py`
- ✅ Admin surface heavily guarded (61 admin refs in `admin_routes.py` — 16 routes)
- ✅ Auth routes guarded (10 current_user refs)
- ⚠️ **Gaps**: cart/order/payment/review routes rely on client-supplied `user_id` path params with no `current_user` dependency (0 refs) → IDOR risk; `SECRET_KEY` hardcoded in `auth_utils.py`; older "parameter-based" auth removed (her fix, noted in report)

## Data Persistence
- ✅ SQLAlchemy + SQLite (`bitebuddy.db`), auto-seeding on startup (Render ephemeral-disk workaround)

## Runnability
- ✅ Backend compiles (8 routers mounted in `main.py`); deployed live (Render + Vercel)
- ⚠️ Committed editor artifacts: `.main.py.swp`, `order_status.py.bak`

## Observations
- Strong, honest team player: frontend cart/menu, backend order/payment, admin status workflows, deployment, and cross-cutting auth/import bug fixes all verifiable in git.
- Real deployment with CORS workarounds and startup seeding — production-minded.
- Same team-level weaknesses: IDOR-prone user-scoped endpoints, hardcoded secret.

## Future Scope
- Add `Depends(get_current_user)` + ownership checks to cart/order/payment/review routes (user_id from token, not path); move SECRET_KEY to env; clean `.swp`/`.bak` files.

---

## Additional Code-Review Findings

- **The order-status endpoint is completely unguarded.** `PATCH /orders/{order_id}/status` in `backend/routes/order_routes.py` sits outside the role-guarded admin router with no authentication at all — any client can advance any order to any status, including "Delivered". Because the same handler auto-issues a loyalty coupon on every third delivered order, this is directly exploitable for coupon farming.
- **The payment flow is a client-controllable simulation.** `payment_callback` accepts an optional `force_result="success"` body field, `call_bkash_gateway()` is `random.random() < 0.8` (a genuine order can randomly "fail"), payment records live in the in-memory `PAYMENTS_DB` dict, and `payment_id` is a `uuid4` truncated to 8 characters — guessable and collision-prone (`backend/routes/payment_routes.py`).
- **A default admin is seeded on every startup with credentials committed to the repo** (`backend/seed_admin.py`: `admin1@bitebuddy.com` / `ChangeMe123!`). The password is bcrypt-hashed at rest, but it is publicly known and the account is recreated automatically on any fresh deploy — change it or gate seeding behind an environment flag.
- **The CORS regex is broader than your deployment.** `allow_origin_regex` in `backend/main.py` permits any origin matching `https://.*\.vercel\.app` with `allow_credentials=True` — every Vercel deployment on the internet, not just yours, can make credentialed cross-origin requests to the API.

## Detailed Feedback (Instructor Review)

**What you did well.** As team lead you owned the widest surface: cart, order, payment, and address routes on the backend, the menu/cart/checkout/order-status frontend, database models and seeding, and deployment to Render and Vercel with CORS configuration and startup auto-seeding to survive Render's ephemeral disk. The admin surface is heavily role-guarded, and your report's claims all verify against git history — including unglamorous cross-cutting fixes: removing legacy parameter-based auth, resolving import bugs, lifting tokens in the frontend. That honesty and follow-through is what a lead does.

**Where to grow.** Your own routes are the app's biggest weakness: cart, order, payment, and review endpoints trust a client-supplied `user_id` path parameter with no authenticated-user dependency — anyone can read or modify anyone else's cart and orders. That is a textbook IDOR in your modules. The SECRET_KEY is hardcoded in `auth_utils.py`, the cart sits in an in-memory dict, and committed `.swp`/`.bak` editor artifacts show sloppy housekeeping.

**Attribution note.** In this three-person team, git credits you with cart/order/payment, admin status workflows, models, seeds, and deployment; auth, coupons, and RBAC were Farhana's. The shallow clone limited some local per-author checks, but the GitHub record is consistent.

**future scope ideas:** derive user identity from the JWT on every user-scoped route, persist the cart, externalize the secret, and clean the repository.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
