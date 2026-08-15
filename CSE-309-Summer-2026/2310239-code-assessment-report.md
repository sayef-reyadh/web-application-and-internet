# Code Assessment Report

**Student:** Yousuf Abdullah
**ID:** 2310239
**Section:** Section 5
**Project:** Online Chess Platform
**Project Type:** Team (with Mohammad Nahiyat Khan 2310003)
**GitHub:** https://github.com/Nahiyat/Web_application_project

---

## ⚠️ Submission Note
No individual submission folder or PDF report was provided for you. This assessment is based on your verifiable commits in the shared team repository (commits authored by `Alif`/`XYZ`, email alifyousufabdullah@gmail.com, GitHub handle Alif-lucifer).

## Tech Stack
- Backend: ✅ FastAPI confirmed
- Frontend: ✅ React confirmed

## Commit History (shared team repo)
| Metric | Value |
|--------|-------|
| Your Commits | 111 (most active committer) |
| Your Active Days | 9 |
| Contributors | 2 |

## Your Contribution (per git attribution)
- **Rankings system (end-to-end):** `ranking_route.py`, ranking schema/controller/service, and the frontend RankingPage.
- **ProfilePage** (frontend).
- **Chess move-rule modules** (game logic).
- **React Router refactor** and **PRD documentation**.

## Security & Authentication
- You wrote no authentication/authorization code.
- Your ranking/leaderboard routes are unprotected (no `Depends()` guard) — the project's auth layer was built by your teammate and not applied to your routes.

## Data Persistence
- Your rankings/profile features are DB-backed (SQLAlchemy) and wired to the frontend ✅

## Observations
- Substantial, real contribution — the most active committer on the project.
- Good adherence to the project's layering conventions.
- Main gap: no security work, and your own API surface is left unguarded.

## Future Scope
- Submit your own project folder and PDF report — grading had to rely entirely on git attribution.
- Protect the routes you build with the existing `Depends(get_current_user)` dependency.
- Contribute to the shared security layer, not just feature code.

## Additional Code-Review Findings

- **Your chess move-rule modules are dead code.** `pawnMoves.js`, `knightMoves.js`, `bishopMoves.js`, `rookMoves.js`, `queenMoves.js`, and `kingMoves.js` (all in `frontend/my-app----frontend/src/`) are imported only by `isCheckmate.js`, `isKingInCheck.js`, and `isStalemate.js` — and **nothing in the app imports those three**. Neither `ChessBoard.jsx` (which sends raw UCI moves over WebSocket to the backend engine) nor `PvCChessBoard.jsx` references any of your move logic, so the rules you wrote never actually validate a move in the running app.
- Relatedly, the special-move modules `castling.js`, `enPassant.js`, and `promotion.js` exist but are never imported anywhere, and `pawnMoves.js` itself implements no promotion or en passant — the rule set is both incomplete and unwired.
- **ProfilePage is orphaned and broken.** `frontend/my-app----frontend/src/pages/ProfilePage.jsx` is not imported by any route or component (no file references it), and it calls `updateUserProfile` from `../services/auth_service` — a function that `auth_service.ts` does not export (it exports only `registerUser` and `loginUser`). Even if routed, the profile update would fail.
- The leaderboard query in `backend/app/controllers/ranking_controller.py` accepts an unbounded `limit` query param (`backend/app/routes/ranking_route.py` defaults to 100 with no maximum), letting any anonymous caller dump the entire users table. And `total_players=len(rankings)` is mislabeled — it returns the page size (capped by `limit`), not the actual number of players, so the response lies whenever more players exist than the limit.
- **Repo hygiene (shared):** 402 `frontend/node_modules/` files are committed to git — dependencies belong in `package.json`, not the repository; add `node_modules/` to `.gitignore` and purge them from history.

---

## Detailed Feedback (Instructor Review)

**Attribution note (team project):** No individual submission folder or PDF was provided, so you were assessed entirely from your verifiable commits in the shared team repo. The evidence is strong: 111 commits over 9 active days make you the **most active committer** on the project (commits authored by `Alif`/`XYZ`, alifyousufabdullah@gmail.com).

**What you did well**
- Your contribution is substantial and real — a rankings system built end-to-end (route, schema, controller, service, and the RankingPage frontend), a ProfilePage, the chess move-rule modules, and a React Router refactor. You follow the project's layering conventions well.

**Where to grow**
- **You wrote no security code, and your own routes are unprotected.** The project has a working auth layer (built by your teammate), but your ranking/leaderboard endpoints don't use it — no `Depends(get_current_user)`. When you build a feature, protecting its API surface is part of finishing it.
- **You submitted nothing individually.** I want to be direct: there is no folder or report in your name, so everything rests on git attribution. If those commits weren't actually yours, there would be nothing to grade. Always submit your own deliverables and commit under an identity clearly tied to you.

Your code contribution is genuinely strong; the gaps are process (no submission) and security (unprotected routes), not ability.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
