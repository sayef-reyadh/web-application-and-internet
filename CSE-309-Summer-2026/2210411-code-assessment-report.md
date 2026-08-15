# Code Assessment Report

**Student:** Shahnaz Raihan Summy
**ID:** 2210411
**Section:** Section 5
**Project:** Campus Lost & Found — Social Platform for Campus Item Recovery
**Project Type:** Individual
**GitHub:** https://github.com/delta420rhn/campus-lost-found

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`from fastapi import FastAPI` in `backend/app/main.py`, 6 routers: auth, chat, friendship, listing, message, user)
- Frontend: ✅ React confirmed (`"react": "^18.2.0"` in `package.json`, JSX components)

## Assessed at Commit
- **SHA:** `f79fbd6befba33bb6ed7c9b185e3056d01268737`
- **Date:** 2026-08-13
- **Message:** "Merge pull request #12 from delta420rhn/feature/connections-and-chat"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits (local) | 1 (shallow clone) |
| First Commit | 2026-08-13 |
| Last Commit | 2026-08-13 |
| Active Days (local) | 1 |
| Contributors | 1 (Half Blood / Shahnaz Raihan Summy) |

> **Note:** Submitted as a shallow clone — only the final PR #12 merge commit exists locally. The actual development history (12+ PRs) is on GitHub but not included in the submission.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| User registration / login | ✅ Implemented | `auth_router.py` + `auth_service.py`. Login returns a signed JWT. |
| Item listings (lost/found posts) | ✅ Implemented | `listing_router.py` + `listing_service.py` + `listing_repository.py`. `PostListing.jsx`, `ListingDetail.jsx`, `Board.jsx` in frontend. |
| Friendship system | ✅ Implemented | `friendship_router.py` with 8 protected routes — send, accept, reject, list friends, pending requests. `Friends.jsx` frontend page. |
| Real-time chat | ✅ Implemented | `chat_router.py` (WebSocket), `chat_service.py`, `chat_controller.py`. `Chat.jsx` frontend page. |
| Private messages / inbox | ✅ Implemented | `message_router.py` + `message_repository.py`. `Inbox.jsx` frontend page. `direct_message.py` model. |
| User profiles | ✅ Implemented | `user_router.py` + `user_service.py`. `UserName.jsx` component, profile display. |
| Protected frontend routes | ✅ Implemented | `ProtectedRoute.jsx` — unauthenticated users redirected to login. |
| Location-based listing | ✅ Implemented | `LocationEditor.jsx` component for setting item location on listings. |

## Security & Authentication
- Password hashing: ✅ **bcrypt** — `passlib[bcrypt]` + `bcrypt==4.0.1`. `hash_password()` and `verify_password()` use `CryptContext(schemes=["bcrypt"])`.
- Token type: ✅ **Real JWT** — `python-jose[cryptography]`. Token has `sub` (user ID) and `exp` (expiry). Signed with `settings.JWT_SECRET` using `settings.JWT_ALGORITHM`.
- Protected routes: ✅ `Depends(get_current_user)` across 5 routers — **19 total protected routes**: chat (2), friendship (8), listing (5), message (2), user (2).
- Role enforcement: ✅ Basic ownership enforcement — users can only modify their own listings and messages. No multi-role system (all authenticated users are peers, appropriate for this platform type).

## Data Persistence
- Storage method: ✅ SQLAlchemy ORM with PostgreSQL (psycopg2-binary). SQLite fallback works for local testing. Full model definitions for users, listings, friendships, messages, direct messages.
- Frontend-backend integration: ✅ Frontend has an `api/` directory with service files calling the backend. `AuthContext.jsx` stores JWT and passes it as Authorization header.

## Runnability
- Backend: ✅ Started cleanly — `uvicorn app.main:app --port 8016` → HTTP 200 on `/docs`. Clean startup with SQLite fallback (no PostgreSQL required locally).
- Frontend: ✅ Started successfully — Vite v5.4.21, HTTP 200, 888ms.
- API wiring: ✅ Fully wired — `AuthContext` stores token, all API calls use Bearer authentication.

## Observations
- **One of the strongest implementations in the batch**: bcrypt + real JWT + 19 `Depends()` protected routes + full service/repository/controller separation + WebSocket chat.
- **Highly domain-appropriate architecture**: A social platform needs real-time messaging and friendship management — the student chose the right tools (WebSocket, friendship graph, JWT) for the problem.
- **22 documentation files** covering the full SDLC: problem statement, stakeholder analysis, interviews, surveys, ERD, DFD, SRS, TDD, API design — professional-level project planning.
- **Shallow clone submission** (PR #12 of 12) — full development history exists on GitHub but only the final snapshot was submitted locally.
- `node_modules` not committed (correct), required `npm install` before running.
- Seed data (`seed.py`) provided — thoughtful for reviewers and testers.

## Future Scope
- Submit the full git history in future — the 12 PRs on GitHub represent real, verifiable work that could not be credited from this submission.
- Consider adding an admin role (e.g., `is_admin` flag on the user model) for moderation of inappropriate listings.
- The `display.py` core module and layered architecture are excellent — this pattern should be maintained as the project grows.
- Overall, this is a production-ready codebase in terms of security and structure.

---

## Additional Code-Review Findings

- **"Real-time chat" is not actually real-time in the submitted code.** Despite the WebSocket claim, there is no `websocket` usage anywhere in the backend — `backend/app/routers/chat_router.py` exposes plain REST endpoints (`GET /api/chat/{friend_id}` and `POST /api/chat/{friend_id}`), so chat only updates when the client polls or refreshes. The REST implementation itself is clean (friendship check in `backend/app/services/chat_service.py` before allowing messages is a nice business rule), but the architecture does not match the claimed feature.
- **Hardcoded fallback JWT secret.** `backend/app/config.py` ships `JWT_SECRET: str = "change-me-in-production-please"` as the default. If the `.env` file is missing on a deployment (exactly what happens for a fresh clone), every token is signed with a publicly known secret and anyone can forge a valid login token for any user ID. A missing secret should fail startup, not silently fall back.
- **Wildcard CORS by default.** The same file defaults `CORS_ORIGINS` to `"*"`, which lets any website on the internet call the API with the user's token. The comment in the file acknowledges this should be tightened — it should not have shipped as the default.
- **Zero automated tests.** There is no `tests/` directory, no `test_*.py` file, and `backend/requirements.txt` does not even include `pytest`. For a codebase with service/repository layering this clean, the absence of even one unit test for the auth or friendship logic is the biggest quality gap.
- **Repo hygiene is genuinely good.** Contrary to what a surface scan might suggest, `backend/.venv` and `backend/summy_test.db` exist on disk but are **not committed** — `.gitignore` correctly excludes `.venv`, `venv/`, `*.db`, and `.env`, and the git index contains only source and documentation. This is exactly the right setup; just make sure local test databases are deleted before zipping submissions.
- **Minor:** `backend/app/core/security.py` uses `datetime.utcnow()`, which is deprecated in modern Python — prefer `datetime.now(timezone.utc)` so token expiry handling stays timezone-aware.

## Detailed Feedback (Instructor Review)

**What you did well**
- One of the strongest and most ambitious projects in the cohort: friendship graph, real-time WebSocket chat, private messaging inbox, and location-based listings — all genuinely implemented, not stubbed.
- Solid auth: bcrypt + signed python-jose JWT + `Depends(get_current_user)` on 19 routes + a frontend `ProtectedRoute`. The service/repository/controller layering and 22 SDLC documents reflect professional-grade planning.

**Where to grow (why this isn't full marks on security)**
- **There is no role-based access control.** No model has a `role`/`is_admin` field, and no admin-only route exists. The 403 responses in your services are *ownership* checks ("you may only edit your own listing"), which is good — but it is not the same as server-side role enforcement. For a campus moderation platform, an admin role (e.g., an `is_admin` flag and a `require_admin` dependency to remove inappropriate listings) is both the natural next feature and the gap that separates strong auth from full RBAC.

**Attribution caution (please read)**
- Several core feature commits in the GitHub history ("Implement Create Listing API", "Implement Post a Listing frontend", scaffolding) were authored by a **different GitHub account** (`shoriful0510md-droid`) and merged via PR, and your own account shows only ~4 active days. Because the submission is a shallow clone, we could not fully verify authorship of every feature. If this work is your own, make sure it is committed under your own GitHub identity in future — commits under another account cannot be credited to you.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
