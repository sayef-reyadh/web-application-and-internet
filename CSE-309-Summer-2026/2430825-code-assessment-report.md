# Code Assessment Report

**Student:** Mohammad Zaid Iqbal Fahad
**ID:** 2430825
**Section:** Section 6 (Group S6-02)
**Project:** Departmental Student on Duty (SoD) Management System
**Project Type:** Team (with Momotaj Akther Happy, 2430798)
**GitHub:** https://github.com/Momotaj-Happy/CSE-309A-WebProject-SoD-Management-System

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`back-end/`, 8 routers)
- Frontend: ✅ React + TypeScript confirmed (Vite)

## Assessed at Commit
- **SHA:** `d16edc0` — 2026-08-12 — "Merge pull request #51 release-v1.0-main-deployment"
- **Repo:** 45 commits, 6 active days (2026-06-17 → 2026-08-12)
- **This student:** 20 commits (`zaid-fahad` 10 + `zaid fahad` 10), 5 distinct days
- **Teammate (Momotaj Happy):** 25 commits, 6 days

## Individual Contribution — verified per-path authorship

| Claimed (report PDF) | Git record |
|----------------------|-----------|
| Auth + RBAC infrastructure (JWT, hashing, auth endpoints, protected routes) | ✅ `routes/auth.py` 1/1, `routes/users.py` 1/1, `services/user_service.py` 1/1, `models/user.py` 1/1, `AuthContext.tsx` 1/1, `services/auth_service.py` 1 of 2 |
| Duty task CRUD + schedule conflict engine (409 on overlap) | ✅ `services/task_service.py` 2/2 (minute-offset interval math), `routes/tasks.py` 2 of 3, `models/task.py` 1/1, `test_task_conflict_engine.py` 1/1 |
| Reporting + swap audit trail (CSV export) | ✅ `routes/export.py` 1/1, `services/export_service.py` 1/1, `test_export_audit_trail.py` 1/1 |
| Faculty/Manager billing UI | ✅ `FacultyBillVerification.tsx` 1/1, `ManagerFinancialApproval.tsx` 1/1 |
| Task assignment UI, user directory, RBAC matrix | ✅ `FacultyTaskAssignment.tsx` 1/1, `UserDirectory.tsx` 1/1, `RBACMatrix.tsx` 1/1 |
| Billing pipeline support + tests | ✅ `routes/billing.py` 1 of 3, `services/billing_service.py` 1 of 3, `test_billing_approval_pipeline.py` 1/1 |
| SRS/TDD docs + own submission report | ✅ `docs/24-project-submission-report.md` 1/1; docs dir shared lead with teammate |

Claims NOT matching git: `SwapAuditTable.tsx` (authored by teammate Momotaj) and `ExportToolbar.tsx` (component does not exist in repo — audit-log/CSV feature exists via backend + dashboard UI). Minor report inaccuracies; core modules are genuinely his.

## Features Found (his scope wired end-to-end)
- Full auth flow (register/login, 4 roles, persistent token storage, protected routes)
- Duty task CRUD with a real **conflict detection engine** — normalized minute-offset conversion + interval-overlap math, 409 on conflicts
- CSV duty report export + swap audit-log endpoints with UI
- Faculty bill verification and Manager financial approval views + RBAC-aware navigation

## Security & Authentication (verified)
- ✅ Real JWT (PyJWT, HS256, exp + iat), HTTPBearer
- ✅ PBKDF2-SHA256 (100,000 iterations) + per-user random salt
- ⚠️ His report claims a `require_role([...])` dependency — **not present in the code**; role checks are inline on a few routes only (managers-only role update, DEPT_MGR delete, Faculty/Manager schedule view)
- ⚠️ Enforcement gaps: bill verify/approve/reject and task CRUD have no role check; `GET /users` and audit-log are token-only; hardcoded `SECRET_KEY` fallback

## Data Persistence
- ⚠️ **All storage is in-memory dicts** (`_USERS_DB`, `_TASKS_DB`, `_SWAPS_DB`, `_AUDIT_LOG_DB`, `_BILLS_DB`) seeded with demo data — no DB, no file persistence; data lost on restart. Report's "SQLAlchemy" claim not present in code.

## Runnability (tested in this session)
- ✅ **26/26 backend tests pass** (incl. his: task conflict engine, billing approval pipeline, export audit trail, auth/users)
- ✅ Backend compiles cleanly

## Observations
- Genuine full-stack contribution: core auth infra, the most complex backend logic (conflict engine), reporting, and 4 faculty/manager UI components are his.
- Same team-level weaknesses as teammate: no persistence, incomplete RBAC enforcement.

## Future Scope
- Replace in-memory stores with a real DB; add a `require_role` dependency that actually guards bill/task routes; remove `SECRET_KEY` fallback from code.

## Additional Code-Review Findings

- **Privilege escalation at registration:** `UserCreate` (in `back-end/api/models/user.py`) inherits `role: UserRole = Field(default=UserRole.STUDENT)` from `UserBase`, and `UserService.create_user` stores `user_create.role` verbatim. A client can `POST /auth/register` with `"role": "DEPT_MGR"` and mint a department manager account — no approval step, no server-side forcing of STUDENT. This is the single most serious gap in your auth module.
- The role guards you do have trust **token claims, not current data**: `update_user_role` and `delete_user` in `back-end/api/routes/users.py` read `payload.get("role")` from the JWT via `get_current_token_payload` and never re-fetch the user. With 24-hour tokens and no revocation, a demoted or deleted manager keeps full manager rights until expiry.
- `DELETE /users/{user_id}` has no self-deletion or last-manager protection — a DEPT_MGR token can delete its own account (or every other manager), leaving the system with no administrator.
- The conflict engine only checks the student's **class schedule and unavailable slots** (`TaskService.check_schedule_conflict` in `back-end/api/services/task_service.py`) — it never compares against the student's other duty tasks, so two overlapping duties for the same student pass with no 409. Also, `_parse_time_minutes` does `int(parts[0]) * 60 + int(parts[1])` with no error handling: a time like `"9:00 AM"` raises ValueError → HTTP 500, and overnight ranges (end < start) are mis-evaluated.
- CSV formula injection: `ExportService.generate_duty_report_csv` (`back-end/api/services/export_service.py`) writes user-controlled fields (`title`, `location`, `log_notes`) straight into cells — a task titled `=HYPERLINK(...)` or `=cmd|...` executes as a formula when the report is opened in Excel. Prefix/escape cells beginning with `=`, `+`, `-`, `@`.
- Password policy is `min_length=6` only (`back-end/api/models/user.py`), and all four seeded demo accounts share `Password123!` — weak credentials guarding a role model with four privilege levels.

## Detailed Feedback (Instructor Review)

**What you did well.** The git record confirms your core modules: the full auth infrastructure (JWT, hashing, protected routes), the duty-task conflict engine with minute-offset interval math returning 409 on overlap, CSV export with the swap audit trail, and four faculty/manager UI components. PyJWT with exp/iat and PBKDF2-SHA256 with per-user salts are properly implemented, and the team's 26 tests — including yours — all pass.

**Where to grow.** Be precise about claims: your report describes a `require_role` dependency that does not exist in the code. Role checks are inline on a few routes only, while bill approval and task mutation routes accept any authenticated user. More fundamentally, everything lives in in-memory dictionaries — there is no database despite a SQLAlchemy claim in the report, so all data vanishes on restart. Also note that SwapAuditTable.tsx was authored by your teammate, not you, and the claimed ExportToolbar component does not exist. Remove the hardcoded SECRET_KEY fallback.

**Attribution note.** Team project with Momotaj Akther Happy. Commit authorship confirms auth/RBAC, the conflict engine, and export are yours; the parser, schedule, swap engine, and billing pipeline are hers.

**future scope ideas:** actually implement `require_role` and guard the billing/task routes, and move to real persistence.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
