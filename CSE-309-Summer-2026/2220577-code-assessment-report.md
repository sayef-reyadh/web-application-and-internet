# Code Assessment Report

**Student:** Gourab Debnath Himel
**ID:** 2220577
**Section:** Section 6
**Project:** TravelTimeAI — AI-Based Travel
**Project Type:** Team (with Momo Karim 2220431)
**GitHub:** https://github.com/Gourab-Himel/traveltimeAI

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`Backend/`)
- Frontend: ✅ React confirmed (Vite + TypeScript, `Frontend/my-app/src/`) — plus an additional static HTML/jQuery app and a Node/Express server (see observations)

## Assessed at Commit
- **SHA:** `4741cd9` — 2026-08-12 — "Final Touch"
- **Repo:** 46 commits, **11 active days** (2026-06-12 → 2026-08-12)
- **This student (`Gourab_Himel`):** vast majority of commits; per-path authorship shows all backend + frontend code files are his
- **Teammate (Momo):** default Vite scaffold `my-chart-app`, `kanban.tsx`, docx files, merge PRs — no feature code

## Features Found
- Register / login (FastAPI + React TS login/register pages with AuthContext + ProtectedRoute)
- Static travel site (Adventure-tourism template style): index/main/trips/trip-single/hotel/blog/contact/about/admin/agent pages
- Trip listings served from a **localStorage DataStore** (`data-store.js`: "Simulates a database using LocalStorage")
- Travel item PUT/DELETE endpoints in FastAPI backed by an **in-memory Python dict** (`travel_items_db = {}`)
- ⚠️ No AI feature found despite the "AI-Based TravelTime" title

## Security & Authentication (verified)
- ❌ Passwords stored **plaintext** (`password = Column(String(100))`, register stores `user.password` directly)
- ❌ Login compares plaintext: `db_user.password != user.password` — **no hashing, no tokens issued**
- ❌ **Zero** `Depends`/auth guards on any endpoint; CORS `allow_origins=["*"]`
- ❌ Registration accepts client-supplied `role` (escalation possible)
- ❌ Hardcoded DB credentials: `mysql+pymysql://root:12345@localhost/traveltimeai` committed to `main.py`

## Data Persistence
- ⚠️ Mixed/partial: users → MySQL (hardcoded localhost creds, not runnable here); travel domain → **in-memory dict** on FastAPI + **localStorage** on frontend; a separate Node/Express + SQLite stack also exists (`server.js`, `database/database.sqlite` committed). No persistent backend for the core domain.

## Runnability
- ⚠️ Backend requires a local MySQL server with root:12345 — **not runnable out of the box**; no tests present

## Observations
- Three parallel stacks (FastAPI, Node/Express, static jQuery) with duplicated purposes — architecture is disorganized.
- Frontend largely a downloaded Bootstrap "Adventure Tourism" template; custom logic confined to data-store.js + auth.
- `Frontend/my-app/.env` and `database/database.sqlite` are committed.

## Future Scope
- Hash passwords (bcrypt/PBKDF2), issue JWTs, add `Depends(get_current_user)` everywhere, drop the hardcoded MySQL password, move travel domain into a real DB, remove the duplicate stacks, implement the claimed AI feature.

---

## Additional Code-Review Findings

- **Split-brain authentication.** The Node/Express stack ([routes/auth.js](repo/Frontend/my-app/routes/auth.js)) hashes passwords with `bcryptjs` into SQLite, while the FastAPI backend ([main.py](repo/Backend/main.py)) stores and compares plaintext in MySQL. The two halves of your own project disagree on how passwords work, and an account created through one stack can never log in through the other.
- **The code itself admits no token is issued.** The Node login route carries the comment `"In a real app, we'd send a JWT token here"` and instead returns the user object, which [public/js/auth.js](repo/Frontend/my-app/public/js/auth.js) drops into `localStorage` — "being logged in," including as admin, is a client-side flag editable in DevTools.
- **The travel endpoints are unreachable dead code.** `PUT/DELETE /api/travel` in `Backend/main.py` operate on a module-level `travel_items_db = {}` with **no create or list endpoint**, so they can only ever return "Item not found"; `item_id` arrives as a query parameter rather than a path parameter, and `updated_data: dict` is mass-assigned via `.update()` — arbitrary field injection if it ever did run.
- **Schema auto-mutation on every boot.** `server.js` calls `db.sync({ alter: true })` at startup, letting Sequelize rewrite the schema automatically — and with `database/database.sqlite` committed, the shipped binary file effectively becomes the source of truth.
- **The same live credential is committed twice.** The hardcoded DSN `mysql+pymysql://root:12345@localhost/traveltimeai` appears in both `Backend/main.py` and `Backend/database.py` — two copies of the database password to keep in sync, and two places it leaks from.

## Detailed Feedback (Instructor Review)

**Tech-stack correction (important):** The required frontend stack for this course is **React**, but your functional frontend is **raw HTML/jQuery**, not React. The working UI is a downloaded Colorlib "Adventure Tourism" Bootstrap template — 19 static `.html` pages using jQuery, OWL carousel, and fancybox, with auth done via `localStorage` redirects. The 6 React files present are an **unwired stub**: `App.tsx` renders only a hardcoded "Welcome" heading, the package.json does not even declare `react`, and the React app is not connected to anything. Because the real, working frontend is vanilla HTML/jQuery, the React requirement was not met.

**What you did well**
- You stood up a working FastAPI backend with MySQL-backed register/login, a React `ProtectedRoute`/`AuthContext` gate, and 46 commits over 11 active days — real, sustained effort.

**Where to grow**
- **Build the real UI in React.** You have a `Login.tsx`/`Register.tsx`/`AuthContext.tsx` that genuinely use hooks and axios — that proves you can do it. The path forward is to reimplement the 19 static template pages as React components and wire them to your FastAPI endpoints, rather than shipping a downloaded template.
- **Security:** passwords are stored and compared in plaintext, registration accepts a client-supplied `role` (privilege escalation), no tokens are issued, and your MySQL password (`root:12345`) is committed to source. Hash passwords, issue JWTs, and never let the client decide its own role.
- **Architecture:** three parallel stacks (FastAPI + Node/Express + static jQuery) with duplicated purpose is disorganized — pick one backend and one frontend. Move the travel domain out of the in-memory dict and localStorage into a real DB.
- **Honesty in scope:** the project is titled "AI-Based" but contains no AI. Implement the claimed feature or remove it from the title.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
