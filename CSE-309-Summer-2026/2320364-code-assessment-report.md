# Code Assessment Report

**Student:** Nowshin Nawar
**ID:** 2320364
**Section:** Section 5
**Project:** AI Resume Analyzer
**Project Type:** Individual
**GitHub:** https://github.com/nowshin-2320364/-https-github.com-ai-resume-analyzer

---

## Tech Stack
- Backend: ✅ FastAPI confirmed (`FastAPI()` in `backend/main.py`, `@app.post/@app.get` routes)
- Frontend: ✅ React confirmed (Vite project, `App.jsx`/`Login.jsx` with hooks)

## Assessed at Commit
- **SHA:** `8dda48f`
- **Date:** 2026-08-11
- **Message:** "Update frontend to use dynamic API URL"

## Commit History
| Metric | Value |
|--------|-------|
| Total Commits | 80 |
| First Commit | 2026-06-12 |
| Last Commit | 2026-08-11 |
| Active Days | 15 |
| Contributors | 1 (Nowshin) |

> Commit history retrieved from the GitHub API (submitted folder contains a snapshot without `.git`). All commits authored by the student alone; development spread across two months.

## Features Claimed vs Found

| Claimed Feature | Status | Notes |
|----------------|--------|-------|
| Drag-and-drop PDF resume upload | ✅ Implemented | Dropzone with drag-over state in `App.jsx`; PDF parsed server-side with `pypdf`. |
| Real-time AI analysis via external API | ✅ Implemented | Groq LLM (`llama-3.3-70b-versatile`) called in `POST /api/upload-resume`; JSON-format response enforced; general analysis fallback when no job description. |
| Dynamic visual match scoring | ✅ Implemented | Match score with color coding (green/amber/red) plus summary, strengths, missing keywords, and suggestions cards. |
| Persistent database of historical scans | ✅ Implemented | SQLAlchemy `SavedAnalysis` model; `POST /api/save-analysis`, `GET/PUT/DELETE /api/history/{id}` — all per-user and auth-protected. |
| User accounts (bonus) | ✅ Implemented | `POST /api/register` + `POST /api/login` with bcrypt hashing. |
| TOTP 2FA (bonus, beyond report) | ✅ Implemented | Full 2FA flow: setup (secret + otpauth URL + QR), enable, and login verification via `pyotp`; QR code rendered in the login UI. |

## Security & Authentication
- Password hashing: ✅ bcrypt via `passlib` (`CryptContext(schemes=["bcrypt"])`)
- Token type: ✅ Real JWT — `python-jose` HS256, 24-hour expiry, `sub` claim
- Protected routes: ✅ `Depends(get_current_user)` via `OAuth2PasswordBearer` on `/api/save-analysis`, `/api/history`, `/api/2fa/setup`, `/api/2fa/enable`
- Role enforcement: ❌ No roles/RBAC (single-user-type app)
- Additional: ✅ TOTP two-factor authentication (setup, enable, verify during login)
- Minor gap: `/api/upload-resume` (AI analysis) is intentionally public — no auth required to analyze a resume; only saving history is protected

## Data Persistence
- Storage method: ✅ SQLAlchemy ORM with SQLite — `users` and `saved_analyses` tables, foreign-key relationship, timestamps
- Frontend-backend integration: ✅ Fully wired — `fetch` calls with `Authorization: Bearer <token>` for all protected endpoints

## Runnability
- Backend: ✅ Python files compile cleanly (`py_compile` passed). Standard FastAPI + SQLAlchemy setup.
- Frontend: Static review only — Vite React app with no third-party runtime dependencies beyond React; API URL resolves dynamically (env var → Codespaces → localhost).
- API wiring: ✅ Frontend calls all 8 endpoints (register, login, 2FA x3, upload-resume, save, history CRUD).

## Observations
- **Strong, self-contained security layer**: JWT + bcrypt + optional TOTP 2FA with a complete QR-code onboarding flow — well beyond the course minimum.
- **Clean AI integration**: PDF text extraction → structured LLM prompt → strict JSON output → rendered result cards with match score coloring and empty states.
- **Single-file backend**: all routes, schemas, and auth helpers live in one 312-line `main.py`. Functional but harder to maintain and review than a modular layout.
- **Hardcoded fallback secret**: `SECRET_KEY` falls back to `"super-secret-key-change-this-in-production"` when `JWT_SECRET_KEY` is unset — fine for dev, risky if deployed without env config.
- No `.env.example` committed (only `.gitignore`), though env vars are properly loaded via `python-dotenv`.
- No secrets committed in the repository.

## Future Scope
- Split `main.py` into routers/controllers/schemas modules as the app grows.
- Make `/api/upload-resume` require authentication (or at least rate-limit it) so the Groq API isn't open to anonymous abuse.
- Add a `.env.example` documenting `JWT_SECRET_KEY` and `GROQ_API_KEY`, and fail fast when they are missing.

## Additional Code-Review Findings

- **The 2FA step is not bound to the password step** (`backend/main.py`): `POST /api/2fa/verify` accepts only `email` + `code` and issues a token without ever re-checking the password — the server has no proof the caller completed factor one. Anyone who obtains a TOTP code (shoulder-surfed within its 30s window, or a leaked secret) gets a full token with just the email address. Tie the verify step to a short-lived server-side "password-verified" state instead.
- **No brute-force or replay protection on TOTP**: the 6-digit code can be attempted unlimited times (no lockout/rate limit on `/api/2fa/verify`), and a code remains reusable within its validity window since used codes are not tracked. There is also no way to disable 2FA and no recovery codes — losing the authenticator means permanent lockout.
- **Bug: intended 400 responses become 500s in `upload_resume`**: the `raise HTTPException(status_code=400, detail="Could not extract text from the PDF.")` sits inside the `try` block, and the blanket `except Exception as e` below it catches `HTTPException` too and re-wraps it as `500 "Analysis failed: ..."`. The broad handler also leaks raw internal/Groq error text to clients via `str(e)`.
- **Prompt-injection exposure in the AI pipeline**: extracted resume text and the job description are interpolated raw into the LLM prompt (`backend/main.py`), so a crafted resume can hijack the instructions or corrupt the JSON output. User PII is also forwarded to a third-party API with no consent notice. Separately, the uploaded file is never validated — no extension, MIME, or size check before feeding it to `pypdf`, inviting parser exploits and memory exhaustion.
- **CORS misconfiguration**: `allow_origins=["*"]` combined with `allow_credentials=True` (`backend/main.py`) — an invalid and dangerous pairing that browsers partially reject; enumerate the real frontend origin instead.
- **Zero automated tests** and weak input validation: the repo contains no test files at all, `UserAuthSchema` types `email` as plain `str` (no `EmailStr`) with no password-strength rule, and `POST /api/register` returns `400 "Email is already registered"` — a user-enumeration oracle that contrasts with the generic 401 used at login.

## Detailed Feedback (Instructor Review)

**What you did well.** This is a well-rounded solo project with a security layer that goes beyond what was required: real `python-jose` JWTs, bcrypt hashing via `passlib`, `Depends(get_current_user)` on the history and save endpoints, and a complete TOTP two-factor flow — secret generation, QR code, enable, and login verification — that many professional apps never implement. The AI pipeline (PDF extraction, structured prompt, strict JSON output, colored match-score cards) is cleanly integrated, and all nine frontend API call sites send proper Bearer tokens.

**Where to grow.** Everything lives in one 312-line `main.py`; that is already hard to review and will not scale. The resume-upload endpoint is public, so anyone can burn your Groq quota anonymously — that is a real exposure, not a nitpick. The fallback secret `"super-secret-key-change-this-in-production"` is dangerous if deployed without env config, and no `.env.example` documents the required variables. There is no role model, though that is defensible for a single-user-type app.

**Submission note.** The submitted folder is a snapshot without `.git`; authorship (80 commits, 15 active days, you alone) was verified against the GitHub API.

**future scope ideas:** modularize the backend, protect the upload endpoint, and fail fast on missing secrets.

---

*This is the final submission for the course. The points under Future Scope are optional ideas should you wish to continue developing this project further. Well done on completing it, and all the best in your future work!*
