# Code Assessment Report

**Student:** Mansura Rashid Rime
**ID:** 2230260
**Section:** Section 5 (Group S5-27)
**Project:** Blood Donation Management System ("শেষ আশা" / Blood Link)
**Project Type:** Team (with Md. Mahedi Hasan Mahin, 2230837)
**GitHub:** https://github.com/mansurarashidrime50/Blood-Donation-Management-System

---

## Tech Stack (team project)
- Backend: ✅ FastAPI (root `backend/`, models/repositories/schemas/services layering, `core/security.py`, `core/websocket.py`)
- Frontend: ✅ React + Vite + Tailwind (root `frontend/`, admin/donor/patient/shared trees)

## Assessed at Commit
- **SHA:** `165fc50` — 2026-08-13 — "Implement donor UPDATE (PUT) API and generate project submission reports"
- **Repo:** 60 commits, 10 active days (2026-06-17 → 2026-08-13), 3 author identities
  - `mansurarashidrime50` (this student): **25 commits**
  - `mahin1710` + `Md. Mahedi Hasan Mahin` (teammate): **35 commits**

## Individual Contribution — as recorded in Git

| Claimed (report PDF) | Git record |
|----------------------|-----------|
| Complete Patient Module (registration, dashboard, blood-request CRUD, search/filter/scheduling) | ❌ NOT attributable — `backend/app/api/endpoints/patient.py` (3/3 commits), `frontend/src/patient/pages/` (3/3), `Patient/` tree (1/1) all authored by **mahin1710** |
| Frontend setup (package.json, App.jsx, Vite) | ⚠️ Initial scaffolding commits only (2026-06-30/07-01) — `Create App.jsx`, `Create package.json` — subsequently replaced by teammate's full implementation |
| Backend setup (requirements.txt, main.py) | ⚠️ Initial stub `main.py` + `requirements.txt` (07-01) — superseded by teammate's layered backend |
| Business analysis docs (01-project-overview, 02-problem-statement) | ✅ Verifiable — authored 2026-06-17, problem statement updated 2026-07-27 |
| Submission report (`docs/Mansura_Project_Submission_Report.md`) | ✅ Verifiable (2 commits in `docs/`) |
| PR management / merging | ✅ Verifiable — 8 merge commits (PRs #9, #13, #49, #50, #63, #64, #72, #73); repo hosted on her account |

Her 25 commits break down as: 8 PR merges, ~12 early scaffolding/web-UI file operations (Create/Delete cycles), 1 docs update, initial business-analysis docs, and the submission report. No patient/donor/admin feature code is authored under her identity.

## Security & Authentication (her code)
- No auth/JWT/bcrypt code authored by her. (Team backend uses JWT + bcrypt per `core/security.py` — authored by teammate; verified in the merged app.)

## Data Persistence (her code)
- No persistence code authored by her. (Team app uses SQLAlchemy + SQLite/PostgreSQL — authored by teammate.)

## Runnability (team app)
- Backend structure is comprehensive (12 models incl. blood_request, donation, meeting, chat, notification, analytics, activity_log; matching/escalation/notification services; websocket). No test suite in the root `backend/` (the teammate's `Donor/` tree has pytest tests for donor flows).
- Committed `backend/blood_donation.db`, duplicated trees (`Admin/`, `Donor/`, `Patient/`, `feature/` alongside root `backend/`/`frontend/`), and `dummy61.txt`/`dummy62.txt` (issue-tracking placeholders) add noise.

## Observations
- **Attribution gap**: the student's PDF report claims the complete Patient Module as her work, but the git history attributes every patient-module file to her teammate. Assessment here follows the git record (same standard applied to other team projects, e.g. Zainab 2331619 and Sadman 2330322).
- Her genuine, verifiable contributions: repo hosting, initial project scaffolding, business-analysis documentation, the detailed submission report, and disciplined PR merging.
- The repo also contains a shared report (`docs/CSE309_Project_Submission_Report.md`) and teammate-specific issue/PR documentation for donor-module bugs.

## Future Scope
- Commit work under one's own Git identity so contributions are attributable; avoid claiming uncommitted code in the report.
- Clean the duplicated trees (`Admin/`, `Donor/`, `Patient/`, `feature/`) and dummy files from the repo root.

## Additional Code-Review Findings

- **`.gitignore` and the tracked database disagree**: the committed `.gitignore` already excludes `*.db`, yet `backend/blood_donation.db` remains tracked — the rule was added without ever running `git rm --cached` on the existing file. As the repo owner who merges every PR, this one-command hygiene fix was within your direct control.
- **Secrets committed in a file you merged**: `docker-compose.yml` ships `SECRET_KEY=production-secret-key-change-it-12345!` and `postgres`/`postgres` database credentials in plaintext, with `restart: always` on every service. This file entered through the PR process you managed — reviewing merges for hardcoded credentials is part of the maintainer role you took on.
- **Editor configuration committed**: `.vscode/settings.json` at the root (plus `.vscode/launch.json`/`tasks.json` inside the `Donor/` and `Admin/` trees) are machine-specific files that the `.gitignore` does not cover and that do not belong in a shared repository.
- **Generated artifacts checked in**: `docs/Mansura_Project_Submission_Report.html` and `docs/CSE309_Project_Submission_Report.html` are generated exports of the `.md` sources living next to them — committing build output alongside source invites divergence between the two.
- **Documentation split across locations**: `02-problem-statement.md` sits at the repository root while its sibling `01-business-analysis/` lives under `docs/` — the numbered business-analysis sequence is broken across two directories, making the doc set harder to navigate than it needs to be.
- **The issue/PR trail covers only your teammate's module**: `docs/` contains six issue write-ups and four pull-request documents — all of them (`issue_1_donor_history_crash.md` through `pull_request_4_backend_profile_sync.md`) document donor-module bugs and fixes. There is no comparable engineering trail for the patient module your report claims, which is consistent with the git attribution discussed above.

## Detailed Feedback (Instructor Review)

**What you did well**
You hosted the repository on your GitHub account, wrote the initial business-analysis documentation (project overview, problem statement), produced a detailed submission report, handled PR reviews and merges with discipline, and set up the early frontend/backend scaffolding.

**Where to grow**
Bluntly: the git history does not support the feature claims in your report. Every patient-module file — the module your report claims as your own work — was committed under your teammate's identity. Your 25 commits break down as PR merges, early scaffolding files later replaced by your teammate's implementation, documentation, and Create/Delete file cycles. None of the donor, patient, or admin feature code carries your authorship. In a team project this is treated as minimal individual code contribution, regardless of intent.

**Attribution note**
If work was done jointly or on a shared machine, it still must be committed under your own account to count as yours. Going forward: write and commit your own feature code, and in reports only claim modules whose files you actually authored. Documentation, repo stewardship, and PR management are genuine skills worth developing, but they do not substitute for implementing features. Next step: in your next project, own one module end-to-end under your own commits.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
