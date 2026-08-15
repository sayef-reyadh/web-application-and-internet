# Code Assessment Report

**Student:** Farhana Ahmed
**ID:** 2312079
**Section:** Section 6 (Group S6-07)
**Project:** BiteBuddy — NextGen Food Ordering Platform
**Project Type:** Team (with Moumita Singh Roy Moon 2312186, Jannatun Naim 2330097)
**GitHub:** https://github.com/moon496/BiteBuddy-NextGen-Food-Ordering-Platform

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (8 routers)
- Frontend: ✅ React confirmed (Vite)

## Assessed at Commit
- **SHA:** `71c37fe` — 2026-08-13
- **Repo:** 91 commits, 20 active days (2026-06-18 → 2026-08-13)
- **This student:** 27 commits (`Farhana Ahmed`), **12 distinct days** (06-19 → 08-12)

## Individual Contribution — verified per-path authorship

| Claimed (report PDF) | Git record |
|----------------------|-----------|
| Authentication system end-to-end (register/login/logout/JWT/bcrypt) | ✅ `auth_utils.py` 1/1, `auth_schemas.py` 3/3, `auth_routes.py` 6 of 8, `authApi.js` 6 of 9, `Login.jsx` 9 of 12, `utils/auth.js` shared |
| Role-based admin access (RBAC) | ✅ `admin_routes.py` 3, `AdminDashboard.jsx` 3 — admin surface role-verified before order/revenue/menu/admin/banned-user management |
| Coupon business logic (best-offer, welcome/loyalty tiers) | ✅ `coupon_routes.py` 1, `Coupon.jsx` 3 of 5 |
| Fraud detection: auto-ban after 3 failed deliveries + manual unban | ✅ admin_routes failed-mark endpoints + banned-users/unban (admin module), documented |
| Account edit/delete with confirmation | ✅ `auth_routes.py` PUT /me, DELETE /me (her PR), Login/account UI |
| Cart popup + menu UI contributions | ✅ `cartpage.jsx` 6, `MenuItems.jsx` 5, `Dashboard.jsx` 3 of 4 |
| SRS/TDD/ERD/system-design docs | ✅ `repo/Farhana/` docs folder (her own docs tree) |

All claims verified against git history. She also reports disciplined issue→branch→PR workflow and evidence-based PR review responses.

## Features (team app — see Moon report for full route list)
- Auth (register/login/logout/me/update/delete) • coupons with personalized tiers • fraud detection (auto-ban at 3 failed deliveries, enforced at login + order tracking) • admin RBAC (orders, revenue, menu, admins, coupons, banned users) • cart/menu/checkout/orders/payments/reviews/addresses

## Security & Authentication (verified)
- ✅ bcrypt + real JWT (PyJWT, HS256, exp) — her implementation
- ✅ Admin routes role-verified (61 admin refs across 16 routes)
- ✅ Ban enforced during login and order tracking with user-facing message
- ⚠️ Team-level gaps: cart/order/payment/review routes accept client `user_id` (IDOR), hardcoded `SECRET_KEY`

## Data Persistence
- ✅ SQLAlchemy + SQLite (`bitebuddy.db`), startup seeding

## Runnability
- ✅ Backend compiles; deployed live (Render + Vercel); Postman-tested endpoints before merge (per report)

## Observations
- Cleanest-reported team role of the three: auth end-to-end, coupon/fraud business logic, and RBAC enforcement are all her verifiable work; she also owned the SRS/TDD/ERD docs.
- Real-world engineering practice: separate bug-fix commits, merge-conflict diff review, verifying branches actually merged.

## Future Scope
- Same as team: token-derived user identity for user-scoped routes; env-based secret.

---

## Additional Code-Review Findings

- **Privilege escalation in your own registration code.** `RegisterRequest` accepts a client-supplied `role` and `register()` writes `role=payload.role` unchecked (`backend/auth_schemas.py`, `backend/routes/auth_routes.py`). Since your `require_admin` guard in `admin_routes.py` only checks `user.role == "Admin"`, anyone can self-register an admin account and take over the admin dashboard. Force `role="User"` at registration.
- **Logout does not survive a restart.** `token_blocklist` in `auth_routes.py` is an in-memory `set` — a server restart re-validates every previously logged-out token (valid for 60 minutes), and the blocklist silently breaks if you ever run more than one worker process.
- **Dead code in the login path.** `login()` calls `create_access_token({"sub": ...})` twice in a row; the first token is discarded. It is harmless but it is exactly the kind of leftover a security-critical path should not carry.
- **Unreachable coupon logic.** In `coupon_routes.py`, `/coupons/my` has a `return` statement before the WELCOME30 block — the new-user public-coupon branch never executes.
- **Discounts computed on a client-supplied subtotal.** `/coupons/best` (and the apply flow) trust `payload.subtotal` from the request body instead of recomputing the cart total server-side — the caller controls the base your percentage discounts are calculated on.
- **CI gives false confidence.** `.github/workflows/backend.yml` runs `pytest || echo "No tests configured yet"` — with zero tests in the repo, the pipeline passes green and triggers the Render deploy on that signal. CI that cannot fail is not a safety net.

---

## Detailed Feedback (Instructor Review)

**What you did well.** Your claimed contributions all verify against git history. You built the authentication system end-to-end — register, login, logout, account update and delete — backed by bcrypt and real PyJWT tokens. The coupon module is genuine business logic: best-offer selection, welcome and loyalty tiers, apply and redeem flows across your six coupon routes. The fraud-detection feature (automatic ban after three failed deliveries, enforced at login and order tracking) is above-average systems thinking; the admin routes are role-verified with 403s. Your issue → branch → PR workflow and evidence-based review responses are how team software should be built.

**Where to grow.** Team-level gaps sit close to your work: the cart and other user-scoped routes trust a client-supplied `user_id` instead of deriving identity from the token — a real IDOR vulnerability in an app whose auth you own — and the SECRET_KEY is hardcoded. Auth this competent deserved better downstream integration.

**Attribution note.** In this three-person team, git credits you with auth end-to-end, coupons, RBAC, fraud detection, and the SRS/TDD/ERD docs; cart, orders, payments, and deployment were Moon's. You are assessed on your own verified work.

**future scope ideas:** drive user identity from the JWT on every user-scoped route, move secrets to the environment, and add tests for your auth and coupon flows.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
