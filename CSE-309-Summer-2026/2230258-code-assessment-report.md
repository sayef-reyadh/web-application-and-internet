# Code Assessment Report

**Student:** Rahie Sakir
**ID:** 2230258
**Section:** Section 5
**Project:** Ashen Keep – 2D Souls-like Action Platformer
**Project Type:** Individual
**GitHub:** https://github.com/Rahie-Sakir/Ashen-keep-2d-game

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`backend/app.py`, `create_app` factory + `backend/database.py` JSON store)
- Frontend: ✅ React confirmed (`"react"` + `"typescript"` + Vite in `frontend/package.json`; HTML5 Canvas game engine in `frontend/src/game/`)

## Assessed at Commit
- **SHA:** `1b05453` — 2026-08-13 — "Merge pull request #20 from Rahie-Sakir/feature-branch"
- **Repo:** 19 commits, 9 active days (2026-06-12 → 2026-08-13), single author (Rahie-Sakir), feature-branch PR workflow

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| 2D action game engine | ✅ Implemented | `frontend/src/game/` — 12 modules (`GameCanvas.tsx`, `world.ts`, `enemies.ts`, `weapons.ts`, `levels.ts`, `render.ts`, `input.ts`, `audio.ts`, `rng.ts`, `types.ts`, `constants.ts`): movement, jump, dodge, parry, light/heavy attacks, weapons, health/stamina, checkpoints, bosses. |
| 3 handcrafted levels + bosses | ✅ Implemented | Moonroot Hollow, The Bellfoundry, Glass Basilica (`levels.ts`); progression with character upgrades, echoes, weapon unlocks. |
| Persistent Save Game (full-stack) | ✅ Implemented | `PUT/GET/DELETE /api/saves/{player_id}` — frontend serializes game state → FastAPI validates/sanitizes → stored in `db.json`; load restores state (validated defaults, `SAVE_VERSION`). |
| Player registration | ✅ Implemented | `POST /api/players` with name validation (2–24 chars) and case-insensitive idempotency. |
| Leaderboard | ✅ Implemented | `GET/POST /api/leaderboard` — only completed 3-level runs with positive time accepted; ranked by fastest time, `?limit=` clamp (1–100). |
| Game screens/UI | ✅ Implemented | `MainMenu`, `GameScreen`, `HowToPlay`, `Codex`, `Leaderboard`, `TopBar` in `frontend/src/screens/`. |
| Health check | ✅ Implemented | `GET /api/health` with uptime timestamp. |

## Security & Authentication
- ❌ **No authentication whatsoever** — no login, no passwords, no tokens, no roles.
- Player identity is a client-supplied `player_id`; anyone can fetch, overwrite (`PUT`), or delete another player's save and view/insert leaderboard entries.
- CORS is wide open (`allow_origins=["*"]`), which is acceptable for a public game but combined with no auth means no access control at all.
- ✅ Positive: the backend treats the client as untrusted for **data validation** — request-size cap (256 KB), save-state sanitization with defaults, numeric coercion with finite checks, and leaderboard submission rules are enforced server-side.

## Data Persistence
- Storage method: ✅ **File system** — `backend/db.json` via thread-safe `JsonDB` (RLock-guarded), atomic writes (temp file + `fsync` + `os.replace`), reload-resilient (`_load` falls back to empty store on corruption).
- Frontend-backend integration: ✅ Fully wired — typed `api.ts` wrapper (98 lines) used by the save/leaderboard/registration screens; no hardcoded data arrays.
- On-disk shape kept identical to the former Express backend (no migration needed).

## Runnability & Tests
- Backend: ✅ All Python files pass `py_compile`.
- Tests: ✅ `backend/tests/test_api.py` — 226 lines, 10 async E2E tests via httpx ASGITransport: health, idempotent registration, invalid names, save round-trip + defaults + delete, unknown player/invalid state rejection, leaderboard ranking + rejection rules, restart persistence, invalid JSON/404 handling.
- Frontend: static review only (TS + Vite; deployable with `/api` proxy or same-origin hosting).

## Observations
- **Serious engineering depth**: the save-state sanitizer (`_sanitize_save_state`), thread-safe atomic JSON persistence, and parameterized route ordering (saves before `{player_id}`) show care beyond typical student work.
- **Honest history**: report openly documents that the project originally had an Express/Node backend that was replaced by FastAPI mid-development — the docs remain partially inconsistent, but code follows the active FastAPI implementation.
- **Outstanding documentation**: 23 numbered DOCS files (PRD, SRS, ERD, DFD, use cases, TDD, API design, etc.) plus README and `requirement.txt`.
- Web-game scope fits CSE309 well: real end-to-end persistence feature (save/load) with meaningful server-side validation.

## Future Scope
- Add at least a lightweight identity mechanism (e.g., device-bound session token) so saves and leaderboard entries can't be trivially modified/deleted by others.
- Add a password/API-key or anonymous-token flow for player accounts if account protection matters.
- Resolve the stale Express-era docs (`DOCS/22-api-design.md`, README backend sections) to match the FastAPI implementation.

## Additional Code-Review Findings

- **Express-era debris is still committed**: `backend/package-lock.json` — a Node lockfile from the replaced Express backend — sits inside the Python backend directory, alongside the root `package-lock.json`. The migration was completed in code but not cleaned in the repository.
- **Runtime data is under version control**: `backend/db.json` (the live player/save/leaderboard store) is committed, so every local test run or play session dirties the working tree and any cloned copy ships with someone else's game data. It belongs in `.gitignore` with a seeded empty template.
- **Every mutation rewrites the entire database**: `_flush_unlocked` in [database.py](repo/backend/database.py) serializes all players, saves, and scores to disk (with `fsync`) on each save upsert or leaderboard post — correct and atomic, but O(total database size) per request, so write cost grows linearly with usage.
- **All lookups are linear scans**: `get_player`, `find_player_by_name`, and `get_save` iterate lists with `next()` — there is no indexing anywhere, so reads are O(n) per request. Fine for a class project, but worth naming as the scaling ceiling of this design.
- **The lock is single-process only**: the `RLock` guards threads inside one Python process; running uvicorn with `--workers > 1` would load a separate `JsonDB` copy per worker and silently split the database — nothing in the code or docs warns about this.
- **Tests cover the API only, not the game**: the 10 tests exercise the FastAPI surface, but the 12 game-engine modules (`enemies.ts`, `weapons.ts`, `levels.ts`, `rng.ts`, …) — the majority of the codebase and all of its logic — have no tests of any kind.

## Detailed Feedback (Instructor Review)

**What you did well.** Although this is a game, it is a legitimate FastAPI + React full-stack application and it meets the stack requirement. You built seven real endpoints — player registration, save/load/delete, and a leaderboard — backed by a thread-safe, atomic JSON store, with a fully wired typed TypeScript client. The server-side engineering is the standout: request-size caps, save-state sanitization with validated defaults, leaderboard submission rules, and a 226-line async end-to-end test suite demonstrate care well beyond typical student work. The honest documentation of the mid-project Express-to-FastAPI migration is appreciated.

**Where to grow.** The absence of any identity mechanism is the defining weakness: player identity is a client-supplied string, so anyone can fetch, overwrite, or delete another player's save, and CORS is wide open. For a course emphasizing secure full-stack design, shipping zero authentication — no passwords, tokens, or sessions — is a significant gap you should be able to explain and fix. Some Express-era documentation also remains stale.

**future scope ideas:** add a lightweight session or token flow binding saves to a verified identity, protect mutating routes, and reconcile the remaining docs with the FastAPI implementation.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
