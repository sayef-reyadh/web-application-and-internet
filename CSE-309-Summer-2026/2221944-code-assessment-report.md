# Code Assessment Report

**Student:** Fardin Abdullah
**ID:** 2221944
**Section:** Section 5
**Project:** Smart Workspace Manager
**Project Type:** Individual
**GitHub:** https://github.com/fardinabdullah/cse_309_project

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`main.py`; 6 routers)
- Frontend: ✅ React confirmed (Vite + React)

## Assessed at Commit
- **SHA:** `1647a79` — 2026-08-12 — "updated design"
- **Repo:** 59 commits, 13 active days (2026-06-17 → 2026-08-12), single author (Fardin Abdullah)
- Deployed: frontend on GitHub Pages, backend on Render (Docker)

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| JWT auth (signup/login) | ✅ Implemented | `routes/auth.py` + `services/auth_service.py`; bcrypt + JWT (30-min expiry), `OAuth2PasswordBearer` on `/auth/login`. |
| Workspace CRUD | ✅ Implemented | `routes/workspace.py` (86 lines, 5 Depends) + service. |
| Paper upload with auto metadata extraction | ✅ Implemented | `routes/paper.py` (337 lines, 9 Depends) + `paper_service.py` (299 lines) — pdfplumber-based extraction of title/authors/year/journal/sections; files stored in `backend/uploads/`. |
| Full-text search across papers | ✅ Implemented | Search endpoints in paper routes + `PaperSearch.jsx`. |
| Interactive PDF reader + progress tracking | ✅ Implemented | `routes/reading.py` (137 lines) + `PaperReader.jsx` with page navigation/zoom/fullscreen and persisted progress. |
| Topic categorization / Knowledge Map | ✅ Implemented | `routes/topics.py` (97 lines) + `KnowledgeMap.jsx`, `TopicRating.jsx`, `TopicCategories.jsx` (Hard/Moderate/Easy). |
| Document management with version control | ✅ Implemented | `routes/documents.py` (272 lines, 7 Depends) + `DocumentDashboard/List/Upload/Viewer`. |
| Dark glassmorphism responsive UI | ✅ Implemented | `styles/` CSS, workspace dashboard (`WorkspaceManager.jsx`). |

## Security & Authentication
- Password hashing: ✅ bcrypt (`passlib`)
- Token type: ✅ Real JWT (`python-jose`, HS256, 30-min expiry)
- Protected routes: ✅ `Depends(get_current_user)` on all routers (7–9 Depends per router)
- ⚠️ Notes: single user role (no RBAC — a personal workspace app, no admin concept); `get_current_user` trusts token claims without re-checking the DB; fallback `SECRET_KEY` value is hardcoded in `utils/security.py` (should come only from env)

## Data Persistence
- Storage: ✅ **MongoDB** via async `motor` driver (`database/mongodb.py`) — `users` + `workspaces` collections (embedded papers/documents arrays); `/test-db` ping endpoint
- Frontend wiring: ✅ Axios API layer (`api/*.js`) with centralized `config.js` base URLs (deployment lesson documented); no hardcoded data

## Runnability
- Backend: ✅ All 22 Python files (excluding committed `.venv`) pass `py_compile`
- Frontend: static review only; deployed live on GitHub Pages

## Observations
- **Real depth**: automatic PDF metadata extraction (pdfplumber) + full-text search + progress-tracked reader + Knowledge Map — a genuinely feature-rich solo project.
- **28 documentation files** (business analysis → test plan → traceability matrix → signoff) — the most complete doc chain seen so far.
- ⚠️ **Repo hygiene issue**: the full `backend/.venv` (hundreds of MB including torch, easyocr, `__pycache__` bytecode) is committed to the repository. This bloats the submission massively and is the main quality concern; it doesn't affect functionality.
- Deployment is real and documented (GitHub Pages SPA + Render Docker + env vars + CORS origins enumerated — better than most).

## Future Scope
- Remove `backend/.venv`, `__pycache__`, and the stray uploaded files from the repo; add a `.gitignore`.
- Keep `SECRET_KEY` env-only (fail fast if unset) and re-verify the user against MongoDB in `get_current_user`.
- Add rate limiting / refresh tokens for longer sessions.

## Additional Code-Review Findings

- **There is no AI in the "Smart" Workspace Manager — and that's okay, but say so.** A full search of `backend/` finds no usage of any AI/ML library (no `openai`, `gemini`, `easyocr`, `torch`, or `transformers` imports, and none in `requirements.txt`). The "auto metadata extraction" is classic deterministic parsing with `pdfplumber` — which is genuinely solid engineering — but the project branding and docs should not imply AI-powered features that do not exist in code.
- **User-uploaded content is committed to git.** `backend/uploads/` contains a real uploaded file (`..._AST402_MidTerm_Report.pdf` — apparently another course's midterm report) and a stray document named `pip install numpy scipy matplotlib.txt`. Upload directories are runtime data; add `backend/uploads/` to `.gitignore` and never commit user content — it bloats the repo and can expose other people's documents.
- **The hardcoded fallback key is a real, high-entropy secret — and it's public.** `backend/utils/security.py` falls back to a full 64-character hex key, and the committed `backend/secret_key.py` shows exactly how it was generated. Because this key is in the repository, the deployed Render instance is fully forgeable on any deploy where the env var is unset — this is worse than an obvious placeholder because nothing looks wrong. Rotate the key (treat it as leaked) and fail startup when `SECRET_KEY` is unset.
- **Documented tests, but no executable tests.** `docs/23-test-plan.md`, `24-test-cases.md`, and `25-traceability-matrix.md` describe an impressive test strategy — yet there is not a single `pytest`/`unittest` reference anywhere in `backend/` or the docs' code samples, and no test file exists. Paper test plans cannot catch regressions; implement even a handful of the documented cases as real tests.
- **Dead and duplicated PDF dependencies.** `requirements.txt` pins `PyPDF2==3.0.1`, `pypdf==3.17.1`, and `pdfplumber==0.10.3`, but only `pdfplumber` is ever imported. `PyPDF2` is officially deprecated in favor of `pypdf` — carrying both (unused) is dependency drift. Also, `pydantic==1.10.13` pins you to the legacy v1 API.
- **Neither `.gitignore` covers `.venv`.** The root `.gitignore` ignores only `node_modules/` and `backend/.gitignore` covers `__pycache__`/`*.pyc`/`.env` — the giant `backend/.venv` in the submission is untracked but *not ignored*, so one careless `git add .` commits hundreds of megabytes. Add `.venv/` explicitly.
- **Minor:** `backend/utils/auth_dependency.py` prints JWT failures with `print("JWT ERROR:", e)` — use the `logging` module so auth errors are captured properly in production log streams instead of stdout.

## Detailed Feedback (Instructor Review)

**What you did well.** This is one of the strongest solo submissions reviewed. You implemented real JWT authentication (python-jose, HS256) with bcrypt hashing and protected every router with `Depends(get_current_user)` — exactly the pattern the course teaches. Beyond that, you delivered genuine depth: pdfplumber-based metadata extraction, full-text search, a progress-tracked PDF reader, and a Knowledge Map. The 28-file documentation chain and the documented, working deployment (GitHub Pages + Render Docker) show professional habits.

**Where to grow.** Two issues need attention. First, repo hygiene: committing the entire `backend/.venv` — hundreds of megabytes including torch and bytecode — bloats the repository and suggests `.gitignore` discipline came late. Second, security details: `get_current_user` trusts token claims without re-checking MongoDB, the fallback `SECRET_KEY` is hardcoded, and there is no role model — the app is single-role by design, which limits multi-user scenarios.

**Submission note.** The submitted folder was a shallow clone, but your documented 59 commits over 13 active days show steady, real development.

**future scope ideas:** clean the repo history, move secrets to environment-only configuration, re-verify users in the dependency, and consider refresh tokens.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
