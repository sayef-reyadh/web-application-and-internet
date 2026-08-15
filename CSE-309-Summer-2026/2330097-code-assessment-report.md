# Code Assessment Report

**Student:** Jannatun Naim
**ID:** 2330097
**Section:** Section 6 (Group S6-07)
**Project:** BiteBuddy — NextGen Food Ordering Platform
**Project Type:** Team (with Moumita Singh Roy Moon 2312186, Farhana Ahmed 2312079)
**GitHub:** https://github.com/moon496/BiteBuddy-NextGen-Food-Ordering-Platform

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (8 routers)
- Frontend: ✅ React confirmed (Vite)

## Assessed at Commit
- **SHA:** `71c37fe` — 2026-08-13
- **Repo:** 91 commits, 20 active days (2026-06-18 → 2026-08-13)
- **This student:** 22 commits (`Mst. Jannatun Naim` 15 + `Yeonali` 7), **6 distinct days** (06-19 → 08-13)

## Individual Contribution — verified per-path authorship

| Claimed (report PDF) | Git record |
|----------------------|-----------|
| Order-tied review system | ✅ `review_routes.py` 2/2, `reviewApi.js` 2/2, `Reviews.jsx` 3 of 4 — her module |
| Delivery address book | ✅ `address_routes.py` 2 of 4, `addressApi.js` 2 of 3, `AddressBook.jsx` 3 of 5 |
| Coupon UI + payment UI integration | ✅ `Coupon.jsx` 2 of 5, `couponApi.js` 1, `Payment.jsx` 1, `paymentApi.js` 1 |
| Menu-items API groundwork + data models | ✅ `model.py` 1, `migrations.py` 1/1, `seed` work |
| Admin dashboard contributions | ✅ `AdminDashboard.jsx` 3 of 9 (incl. conflict-resolution revert documented) |
| Business analysis phase docs | ✅ early-phase docs (project overview, interviews, surveys) per report |

Her claims are verified: she is the lead of the review module and a co-owner of addresses; remaining work is shared/assist on coupons, payments, admin, and models.

## Features (team app — full route list in Moon's report)
Reviews (order-tied, no anonymous/duplicate), address book, coupons, payments, cart, orders, admin dashboard, menu, auth — 8 routers, 40+ routes, live deployment.

## Security & Authentication (team-level, verified)
- ✅ bcrypt + real JWT, admin routes role-guarded
- ⚠️ User-scoped routes accept client `user_id` (IDOR risk); hardcoded SECRET_KEY

## Data Persistence
- ✅ SQLAlchemy + SQLite, startup seeding

## Runnability
- ✅ Backend compiles; deployed (Render + Vercel)

## Observations
- Solid supporting role: owns the review system end-to-end, co-owns address book, contributed early docs + models + migrations.
- Honest, well-scoped report; demonstrates careful merge-conflict discipline (revert-to-known-good then re-apply).
- Scope is lighter than teammates (22 commits vs 27/42) and mostly shared authorship — reflected in the features score.

## Future Scope
- Same as team: token-derived user identity for user-scoped routes; env-based secret.

## Additional Code-Review Findings

- **Your ownership checks enforce against a self-asserted identity.** `add_review` compares `order.user_id` to `payload.user_id` — a client-supplied JSON body field — and no token is decoded anywhere in `backend/routes/review_routes.py`. The same applies to `DELETE /reviews/{review_id}?user_id=...`, which takes identity as a query parameter. The order-tied guarantee is only as strong as the caller's honesty until identity comes from the JWT.
- **Duplicate-review prevention is application-level only.** The `Review` model in `backend/model.py` has no `UNIQUE(order_id, item_id)` constraint, so two concurrent submissions can both pass the existence check and insert duplicates. Add the database constraint as the real backstop.
- **N+1 query pattern in your reviewable-items endpoint.** `GET /reviews/reviewable/{user_id}` issues one `OrderItem` query per order inside a Python loop — fine at demo scale, but it will degrade as order history grows; a single join or `selectinload` would fix it.
- **Average rating is computed in Python.** `GET /reviews/{item_id}` loads every review row into memory and does `sum(rating)/len(reviews)` instead of SQL `AVG`/`COUNT`, which will not scale with review volume.
- **No automated tests exist anywhere in the repository** (no test files under `backend/` or `frontend/`), so your module's 403/400 paths and duplicate guard are verified only by manual use.
- **Positive detail:** `_serialize` deliberately omits `user_id` from public review payloads, so the public listing does not leak reviewer identity — a good privacy instinct worth keeping.

## Detailed Feedback (Instructor Review)

**What you did well.** Your review module is the most clearly-owned piece of the BiteBuddy codebase: all four review routes, including the order-tied "reviewable" check that blocks anonymous and duplicate reviews, plus the frontend wiring. You also co-own the address book and contributed the early-phase documentation. Your report was honest about scope, which counts.

**Where to grow.** Your footprint is noticeably narrower than your teammates' — roughly half their commit volume, and much of it shared authorship on coupons, payments, and admin. The review module is correct but conventional; it does not compensate for thin coverage elsewhere. Team-level security issues touch your code too: user-scoped routes trust a client-supplied user ID (an IDOR flaw), and the cart lives in memory. You must be able to explain and defend every team decision, not just your module.

**Attribution note.** Team project with Moumita Singh Roy Moon and Farhana Ahmed. The local clone is shallow, so per-author attribution relies on the remote Git history; on that record your claims check out, but your overall share is the smallest of the three.

**future scope ideas:** take sole ownership of one more full vertical slice, derive user identity from the token instead of the client, and be ready to walk through your teammates' modules.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
