# Code Assessment Report

**Student:** SM Nafi
**ID:** 2220947
**Section:** Section 6
**Project:** MeatTech — Smart Inventory & Supply Chain
**Project Type:** Team (with Md Sumon Islam 2221333)
**GitHub:** https://github.com/md-sumon-islam/MeatTech

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`New folder/.../backend/main.py`, `models.py`)
- Frontend: ✅ React confirmed (Vite, `New folder/.../frontend/`)

## Assessed at Commit
- **SHA:** `693a386` — 2026-08-13 (repo HEAD)
- **Repo:** 30 commits, 10 active days
- **This student (`NAFI-SM`):** 06-20 docs phase 3–4 (`f8ed76d`, `01803ca`); 07-02 base setup (`4729d3b`); 07-22 FastAPI backend + review system (`99a3fbc`); 07-30 frontend+backend code (`68cc526`) + full Vite frontend with UI styling/CRUD (`6f82d23`, 19 files); 08-06 merge-conflict resolution + doc restructure (`1a27edd`); 08-12 login/logout + dashboard updates (`bddb60f`, 8 files); plus merges (07-30, 08-12, 08-13)

## Features Found (his contributions)
- FastAPI review API (GET/POST/PUT/DELETE `/api/reviews` with Pydantic schema + id counter)
- Vite React frontend: Dashboard.jsx, App.jsx, index.html, login/logout UI, CRUD styling (ProductReview.jsx shared with teammate)
- Documentation phases 3–4

## Security & Authentication (verified)
- ❌ **None.** Review API has no hashing/tokens/guards; CORS `allow_origins=["*"]`
- ❌ His "login/logout" addition is frontend-UI-only — no authenticated backend endpoint

## Data Persistence
- ❌ Review data stored in an **in-memory Python list** (`reviews_db: List[Review] = []`) — lost on every restart

## Runnability
- ⚠️ No tests; `__pycache__/*.pyc` files committed (including in his own commits)

## Observations
- Real but shallow full-stack slices; shared repo is cluttered (nested `New folder/` tree, pyc artifacts, stray files). Contribution split with teammate: Nafi built the initial FastAPI reviews backend + frontend styling/dashboard/login UI; Sumon added the review API endpoint polish + ProductReview component + final root upload.

## Future Scope
- Move review store to a persistent DB; add real authentication; clean repo artifacts; add tests.

---

## Additional Code-Review Findings

- **Your login error message hands attackers the credentials.** In `New folder/MeatTech-sumon-doc-phase1/frontend/src/Dashboard.jsx`, a failed login renders `❌ Wrong Phone or Password! Use 12345 / 12345` — the UI itself discloses the valid credential pair, so the hardcoded check is not even a secret.
- **No rating validation in your review schema.** `New folder/.../backend/main.py` declares `rating: int` with no range constraint — the API accepts `-50` or `9999` even though your own UI promises a 1–5 scale. Use `Field(ge=1, le=5)`.
- **Unsafe global id counter.** `review_id_counter` is module-level mutable state incremented without any lock; under concurrent POSTs (FastAPI runs sync handlers in a thread pool) two requests can read the same counter value and produce duplicate ids.
- **Your backend cannot be installed from its own dependency file.** Your FastAPI backend folder has no `requirements.txt` at all, and the repo's only requirements file (`backend/requirements.txt`) lists `django`, `djangorestframework`, `django-cors-headers` — no `fastapi` or `uvicorn`. `pip install -r requirements.txt` produces an environment that cannot run your code.
- **The entire storefront is hardcoded client-side state.** In `Dashboard.jsx`, the product catalog, cart, coupon logic, checkout, and order history are all local React state — checkout "places" orders that are never sent to any API and vanish on page refresh. Only the review widget touches the backend.

---

## Detailed Feedback (Instructor Review)

**Attribution note (team project):** Graded on your individual contribution only. Per-commit authorship shows you built: the initial FastAPI reviews backend (GET/POST/PUT/DELETE `/api/reviews`), the Vite React frontend styling/dashboard/login UI, and documentation phases 3–4. The `ProductReview` component and review-endpoint polish were your teammate's.

**What you did well**
- You produced a genuine full-stack slice — a FastAPI review API with a Pydantic schema, and a React frontend with a dashboard, login/logout UI, and CRUD styling — across 7 active days with a PR workflow.

**Where to grow**
- **Your login is frontend-only.** The dashboard is gated by a hardcoded credential check in `Dashboard.jsx` (`loginPhone === '12345' && loginPassword === '12345'`). That is real frontend protection (and it's why this isn't scored as zero), but anyone can bypass it by calling the API directly — there is no backend authentication at all. Real auth means the *server* verifies a token, not the browser comparing two constants.
- **No persistence:** reviews are stored in an in-memory Python list (`reviews_db = []`) and vanish on every restart. Persist them to SQLite/MySQL.
- **Scope vs. title:** the project is called "Smart Inventory & Supply Chain" but implements only a review module. The core inventory/supply-chain domain is unbuilt.
- **Repo hygiene:** `__pycache__/*.pyc` files and nested duplicate trees (`New folder/`) are committed — clean these up.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
