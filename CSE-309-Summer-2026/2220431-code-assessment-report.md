# Code Assessment Report

**Student:** Momo Karim
**ID:** 2220431
**Section:** Section 6
**Project:** TravelTimeAI — AI-Based Travel
**Project Type:** Team (with Gourab Debnath Himel 2220577)
**GitHub:** https://github.com/Gourab-Himel/traveltimeAI

---

## Tech Stack
- Backend: FastAPI confirmed (authored by teammate)
- Frontend: React confirmed (authored by teammate)

## Assessed at Commit
- **SHA:** `4741cd9` — 2026-08-12 (repo HEAD)
- **Repo:** 46 commits, 11 active days
- **This student:** files touched: `my-chart-app/` (default Vite scaffold), `kanban.tsx`, 4 `.docx` documents, repo-root `index.html`, 1× `Backend/requirements.txt`, 1× `Frontend/my-app/package.json` — plus 12+ merge PR commits (#20–#40)

## Individual Contribution — verified via GitHub API
- **`my-chart-app/`** — the stock Vite React template (boilerplate App.tsx/main.tsx/assets) — **removed from the final tree**
- **`kanban.tsx`** — a kanban mock component — **removed from the final tree**
- **Documents** — committed at repo root: AI-Based TravelTime – Project Overview.docx, Product Requirement Document.docx, Software Requirements Specification.docx, Technical Design Document (TDD).docx (these remain in the repo)
- No commits on any file that survives in the final codebase (all backend `Backend/*`, `Frontend/my-app/src/*`, `public/*`, `server.js`, `routes/*`, `database/*` belong to Gourab_Himel)

## Claims vs. Git History
Her PDF claims: "AI Core Integration & Backend Architecture — designed and implemented POST /api/v1/recommendations with generative AI", "interactive destination search", "real-time budget tracking data-visualization (frontend/feature/data-vis)", DTO/DFD architecture.
**Verifiable in repo: none of this exists.** A repository-wide search finds no AI/recommendation code, no `/api/v1/recommendations` endpoint, no `frontend/feature/` directories — in the current tree or anywhere in the 46-commit history.

## What She Did Contribute
- 4 project documents (overview/PRD/SRS/TDD docx files)
- Transient scaffold/mock files (later removed)
- PR/merge management on the shared repo

## Verdict
Documentation and workflow support only; the extensive AI/backend/frontend contributions claimed in the report are not present in the repository.

---

## Additional Code-Review Findings

Findings below concern the repository you submitted and claimed features in; per-file history attributes this code to your teammate, but as the submitting team member you share responsibility for the state of the repo.

- **Zero tests in the entire submission.** No `tests/` directory and no `*.test.*`/`*.spec.*` file exists anywhere in the repo — despite the Technical Design Document you committed describing a testing strategy.
- **Environment file and live database committed.** `Frontend/my-app/.env` (hardcoded `VITE_API_BASE_URL`) and `Frontend/my-app/database/database.sqlite` (a binary database with live user rows) are both in version control, and the committed `.gitignore` excludes neither.
- **The Node server auto-mutates its schema on every boot.** `Frontend/my-app/server.js` runs `db.sync({ alter: true })` at startup — Sequelize will rewrite the production schema automatically, which can silently alter or drop columns.
- **The only travel-domain endpoints are dead code.** In `Backend/main.py`, `PUT/DELETE /api/travel` operate on a module-level `travel_items_db = {}` that has **no create or list endpoint**, so both routes can only ever return "Item not found" — and `updated_data: dict` is mass-assigned via `.update()` with no validation.
- **Template and tooling residue was never cleaned.** The submission includes `Frontend/my-app/prepros-6.config`, the Colorlib template's `readme.txt`, font license PDFs, and a malformed image name (`Screenshot 2025-11-27 213504.png1.png`) — none of it belongs in a final submission.

## Detailed Feedback (Instructor Review)

**Attribution note (team project):** You were graded only on work verifiably authored by you, per the team-project policy.

**What we could verify**
- 4 project documents (Project Overview, PRD, SRS, TDD `.docx`) committed at the repo root — these remain in the repo and are your clearest contribution.
- A default Vite scaffold (`my-chart-app/`) and a `kanban.tsx` mock — both were **removed** from the final tree, so no surviving code is yours.
- PR/merge management on the shared repo.

**The critical problem — claims do not match the repository**
- Your report claims substantial technical work: an "AI Core" (`POST /api/v1/recommendations` with generative AI), interactive destination search, and a real-time budget data-visualization frontend. **A full search of the repo and all 46 commits finds none of this** — no recommendation endpoint, no AI code, no data-viz feature. Writing claims in a report that the code does not support is a serious academic-integrity concern. Only claim what you have actually built and can point to in the repository.

**Where to grow**
- Contribute real, surviving code under your own GitHub account. Right now, nothing you coded remains in the final product.
- Align your report with reality: describe the features that actually exist and that you actually wrote.
- Documentation alone, without a code contribution, does not demonstrate the full-stack skills this course assesses.

---

*This is the final submission for the course. The points under "Where to grow" above are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
