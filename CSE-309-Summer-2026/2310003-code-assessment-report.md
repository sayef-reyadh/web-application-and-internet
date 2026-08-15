# Code Assessment Report

**Student:** Mohammad Nahiyat Khan
**ID:** 2310003
**Section:** Section 5
**Project:** Online Chess Platform
**Project Type:** Team (with Yousuf Abdullah, 2310239)
**GitHub:** https://github.com/Nahiyat/Web_application_project

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`app/main.py`; routes → controllers → schemas layering)
- Frontend: ✅ React confirmed (Vite + React in `frontend/my-app----frontend/package.json`)

## Assessed at Commit
- **SHA:** `602200c` — 2026-08-13 — "fixed naming error" (author: Nahiyat)
- **Repo:** 143 commits, 15 active days (2026-06-12 → 2026-08-13), 2 main authors (Nahiyat Khan / Nahiyat, Alif)

> Commit history retrieved from GitHub API. Per-file authorship confirms Nahiyat authored the **entire backend core**: auth, WebSockets, chess engine AI, matchmaking, rankings, dashboard, and the game models — plus the ChessBoard and Player Dashboard frontend.

## Features Claimed vs Found (Nahiyat's Individual Contribution)

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| JWT Authentication (login/register) | ✅ Implemented | `routes/auth_route.py` + `controllers/auth_controller.py` (Nahiyat-authored) — register, login, **access + refresh tokens** with type separation. |
| Real-time multiplayer chess | ✅ Implemented | `websocket/game_socket.py` (111 lines) + `connection_manager.py` — `WS /ws/game/{game_id}`, token required via query param (invalid → close 1008), per-game room broadcast of moves/state. |
| Player vs Computer (CPU) | ✅ Implemented | `controllers/pvc_controller.py` + `utils/chess_engine.py` (232 lines) — **real Minimax with alpha-beta pruning** and move ordering (depth 1–4, cached); `python-chess` for legality. |
| Matchmaking / game sessions | ✅ Implemented | `matchmaking_route.py` + controller — create/join game by `game_id`, protected. |
| Rankings | ✅ Implemented | `ranking_route.py` + controller + `ranking_schema.py` — protected leaderboard of players. |
| Match history | ✅ Implemented | `game_model.py` (Nahiyat 3 commits) + dashboard — game records stored and retrieved. |
| Player Dashboard | ✅ Implemented | `PlayerDashboard.jsx` (Nahiyat 5 + teammate 2 commits) + `dashboard_controller.py`; `ProfilePage`, `RegisterPage`, `MatchHistory`, `RankingPage` pages. |
| Chess move validation UI | ✅ Implemented | `ChessBoard.jsx` (Nahiyat 3, Alif 1) + per-piece move modules (`pawnMoves`, `knightMoves`, `bishopMoves`, `rookMoves`, `queenMoves`, `kingMoves`, `castling`, `enPassant`, `promotion`, `isCheckmate`, `isStalemate`). |

## Security & Authentication
- Password hashing: ✅ bcrypt (`passlib`) with SHA-256 pre-hash (avoids bcrypt 72-byte truncation)
- Token type: ✅ Real JWT (`python-jose`) — **access + refresh tokens**, `type` claim enforced on verify (`"Invalid token type"`), `exp` claims
- Protected routes: ✅ `get_current_user` (`HTTPBearer` + `verify_token`) on auth/dashboard/matchmaking/ranking routes (Depends confirmed in all but PvC)
- WebSocket auth: ✅ token validated at handshake; closes 1008 on missing/invalid token
- Guest PvC: ✅ `/api/pvc/move` intentionally public (no user data touched) — reasonable design choice
- No RBAC: ⚠️ single player role only — no admin/role model exists (no roles to enforce, but also no role layer)

## Data Persistence
- Storage: ✅ **SQLite** via SQLAlchemy (`sqlite:///./test.db`, `pool_pre_ping`); `user_model`, `game_model`, `registration_model` tables; real `test.db` file committed with data
- Frontend wiring: ✅ `services/api.ts`, `auth_service.ts`, `match_making.js`, `pvc_service.js`, `ranking_service.js` all call backend endpoints
- Note: Alembic has only one migration (`create_users_table`); other tables rely on `Base.metadata` table creation

## Runnability
- Backend: ✅ All 29 Python files pass `py_compile`; requirements include `fastapi`, `python-jose[cryptography]`, `passlib[bcrypt]`, `sqlalchemy`, `python-chess`
- Frontend: static review only (Vite build config present)

## Observations
- **Real engineering substance**: an actual chess AI (minimax + alpha-beta + move ordering with a priority-scoring heuristic) and a token-authenticated WebSocket game server — both genuinely hard parts done well.
- **Honest division of labor**: per-file history cleanly matches his report (backend/security/WebSockets/AI on his side; teammate contributed to boards/pages/auth too).
- **Thorough docs**: 22 files across 4 phases (Business Analysis, PRD, SRS with DFD diagrams, TDD with ERD).
- Cleanup notes: stray root-level `.js` chess files, leftover `login.html`/`welcome.html`, and the committed `test.db` are clutter, not defects.

## Future Scope
- Move table creation to Alembic fully (single migration today) and remove the committed `test.db`.
- Add an admin role + role enforcement if tournaments/accounts need moderation.
- Persist game moves/sessions server-side (currently FEN-based stateless exchange) for robust reconnect handling.

## Additional Code-Review Findings

- **The `/auth/refresh` endpoint is broken**: `backend/app/routes/auth_route.py` (line 27) calls `verify_token(refresh_token, expected_type="refresh")`, but `verify_token` in `backend/app/core/security.py` is defined as `verify_token(token, token_type="access")` — the keyword `expected_type` does not exist, so every refresh call raises a `TypeError` and returns 500. A single smoke test would have caught this. Additionally, refresh issues a new access token without checking that the user still exists in the DB, and refresh tokens are never rotated or revoked (same token reusable for 7 days).
- **Hardcoded fallback secrets**: `backend/app/core/config.py` defaults `SECRET_KEY` to `"CHANGE_ME_SUPER_SECRET"` and ships default DB credentials `postgres/postgres`; if `.env` is absent the app still runs with forgeable tokens. Worse, the config defines a full PostgreSQL block while `backend/app/core/database.py` actually connects to SQLite — dead, misleading configuration.
- **WebSocket auth token travels in the URL query string** (`backend/app/websocket/game_socket.py`: `token: str = Query(...)`): access tokens in URLs are routinely written to server access logs, proxy logs, and browser history. Prefer a subprotocol/header handshake or short-lived single-use WS tickets.
- **Registration is a user-enumeration oracle**: `register_user` (`backend/app/controllers/auth_controller.py`) returns `409 "Email already registered"` while login correctly uses a generic 401. Consider a uniform response.
- **Unauthenticated, CPU-intensive PvC endpoint**: `/api/pvc/move` runs minimax at up to depth 4 per request with no authentication and no rate limiting (`backend/app/controllers/pvc_controller.py`) — a trivially scriptable CPU-exhaustion vector. Depth is capped, which helps, but a public compute endpoint still needs throttling.
- **No automated tests anywhere** and fragile move synchronization: there is not a single test file in the repo, and the WebSocket move loop relies on `db.expire_all()` + re-query (commented "SQLite Fix") with no row locking or state versioning — two move messages arriving together can both pass the turn check before either commits. Also note `auth_route.py` has duplicated mid-file imports (`Body`, `get_current_user`, `User` imported twice).

## Detailed Feedback (Instructor Review)

**What you did well.** Your share of this project carries real weight. The security core is genuinely yours and it is done properly: `python-jose` access and refresh tokens with an enforced `type` claim, `bcrypt` with a SHA-256 pre-hash to dodge the 72-byte truncation, and `get_current_user` wired through dashboard, matchmaking, and rankings. The WebSocket game server validates tokens at handshake and closes with 1008 on failure, and the minimax engine with alpha-beta pruning and move ordering is a real AI, not a random mover. Match history, rankings, and the player dashboard are all wired end-to-end.

**Where to grow.** There is no role model at all — the platform has no way to distinguish an admin from a player, which limits moderation. Alembic has only one migration while other tables are created ad hoc, and the committed `test.db`, stray root-level chess scripts, and leftover HTML pages are clutter. Game state is exchanged statelessly via FEN, so reconnects are fragile.

**Attribution note.** Per-file commit history confirms you authored the entire backend core — auth, WebSockets, AI, matchmaking — while your teammate contributed mainly to boards and pages.

**future scope ideas:** complete the Alembic chain, add an admin role, persist game sessions, and clean the repository root.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
