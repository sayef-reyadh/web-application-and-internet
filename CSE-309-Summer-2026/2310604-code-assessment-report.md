# Code Assessment Report

**Student:** Ridwan Hasan Khandakar
**ID:** 2310604
**Section:** Section 6
**Project:** VERA — Volunteer Emergency Response Alliance
**Project Type:** Team (with Md. Mahmudul Hasan 2311960)
**GitHub:** https://github.com/MahmudulHasanJoy/VERA

---

## Tech Stack
- Backend: FastAPI confirmed (authored by teammate)
- Frontend: Next.js (React) confirmed (authored by teammate)

## Assessed at Commit
- **SHA:** `553e922` — 2026-08-13 (repo HEAD)
- **Repo:** 58 commits, 12 active days
- **This student:** **5 commits, 2 days** (2026-06-17 → 2026-06-18)

## Individual Contribution — verified via GitHub API
All 5 of Ridwan's commits:
| Commit | Date | Content |
|--------|------|---------|
| `c0e0a32` | 06-17 | README introduction line |
| `f38151c` | 06-18 | Added root `SRS/` + `TDD/` documentation set |
| `9cad331` | 06-18 | Author name update in SRS/TDD docs |
| `d03f801`, `c2b77b8` | 06-18 | Merge PRs #6/#7 (`docs/srs-tdd-fix`) |

Per-path authorship scan confirms **zero** commits by Ridwan on any code: not on frontend pages (shelters/donations/volunteers claimed in his PDF are 100% Mahmud's), not on `lib/api.ts`, not on backend modules, not on `tests/`, not on deploy configs. His root `SRS/` + `TDD/` docs are independent content (differs from teammate's `docs/srs` + `docs/tdd`).

## Claims vs. Git History
His PDF reports: "Frontend & QA contributor… implemented and refined selected React pages (shelters, donations & campaigns, volunteers), client-side validation, API request payloads, manual testing, ~30% effort split."
**Verifiable in git:** documentation only. The claimed frontend work, API wiring, and testing activity do not appear in the repository history for his GitHub identity (or any other identity — those files belong to Mahmud's commits).

## What He Did Contribute
- Author of the root `SRS/` and `TDD/` documentation set (functional/non-functional requirements, use cases, DFD, SRS doc, ERD, system design, database design, API design) — substantive requirements & design documentation.

## Verdict
Legitimate but documentation-only teammate contribution on a strong project. The report's claimed 30% code effort is not supported by the repository history.

---

## Additional Code-Review Findings

- **Your "authoritative" API reference is stale.** `TDD/22-api-design.md` declares itself "the authoritative reference for every endpoint … as implemented in `backend/app/api/routes`", yet it documents no password-reset endpoints (`/auth/forgot-password`, `/auth/reset-password` both exist in the shipped code) and no `/assistant/chat` route. Your documents were last touched on 06-18 while the code evolved through 08-13 and were never reconciled.
- **The flagship feature is absent from your requirements.** The shipped product's headline capability — the Gemini-powered VERA Bot assistant — appears nowhere in your `SRS/` or `TDD/` set (no mention of an assistant, chatbot, or AI anywhere in your ten documents). The requirements and design documentation you own does not describe the product that was actually built.
- **Two competing documentation trees with no disambiguation.** Your root `SRS/` + `TDD/` folders sit beside your teammate's `docs/srs/` + `docs/tdd/` — two parallel, divergent document sets with no README or cross-reference telling a reader which one is current.
- **No QA artifact despite a claimed QA role.** Your report claims manual testing and QA contribution, but your commits contain no test plan, test cases, or QA evidence of any kind — the repo's only tests (`backend/tests/`, authored entirely by your teammate) are unreferenced by anything you wrote.

---

## Detailed Feedback (Instructor Review)

**Attribution note (team project):** You were graded only on work verifiably authored by your GitHub identity. The full 58-commit history was checked, and all 5 of your commits (over 2 days) are documentation: a README line, the root `SRS/` + `TDD/` document set, an author-name edit, and two doc merges. **Zero commits by you touch any code** — not the frontend pages, not `lib/api.ts`, not the backend, not tests.

**The critical problem — your report does not match the repository**
- I need to be direct with you: your PDF claims you "implemented and refined" the shelters, donations/campaigns, and volunteers React pages, did client-side validation and API payload work, and contributed ~30% of the effort. **None of this appears anywhere in the git history** — every one of those files belongs to your teammate's commits. The gap between what you claimed and what exists is something I am aware of and take seriously. Only claim work you have actually done and can point to.

**What you did contribute**
- The root `SRS/` + `TDD/` documentation set (requirements, use cases, DFD, ERD, API design) is substantive and is real work.

**Where to grow**
- Contribute actual code under your own account. Documentation alone does not demonstrate the full-stack skills this course assesses.

---

*This is the final submission for the course. The points under "Where to grow" above are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
