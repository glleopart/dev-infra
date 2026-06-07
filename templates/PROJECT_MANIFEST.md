# Project Manifest
*Last updated: [DATE] — update after every build session.*
*Auto-generate with: `python scripts/generate_manifest.py --update`*

---

## Project overview

**Project name:** [name]
**Description:** [one sentence — who uses it, what problem it solves]
**Current version:** v0.0
**License:** [MIT / Apache 2.0 / GPL / Proprietary / TBD]
**Tech stack:** [e.g. Python, FastAPI, React, MongoDB]
**Privacy model:** [public OSS / private internal / commercial SaaS]
**AI/ML layer:** [Yes — version vX.Y / No]
**Repository:** [GitHub URL or "private"]

---

## Tech stack detail

| Layer | Technology | Version |
|-------|-----------|---------|
| Language(s) | | |
| Backend framework | | |
| Frontend framework | | |
| Database | | |
| ORM / driver | | |
| Auth | | |
| Payments | | |
| Testing | | |
| Infra / deploy | | |
| AI/ML (if applicable) | | |

---

## File tree

```
project/
├── backend/
│   └── ...
├── frontend/
│   └── ...
├── docs/
│   ├── PROJECT_MANIFEST.md  ← this file
│   ├── SESSION_HANDOFF.md
│   ├── AUDIT_PROMPTS.md
│   └── AUDIT_REPORT.md
├── scripts/
│   ├── audit_pipeline.py
│   └── generate_manifest.py
├── .env.example
├── .gitignore
└── README.md
```

---

## File registry

*One entry per source file. Add every new file here immediately after creation.*

### `backend/...`
- **Lines:** 
- **Purpose:** 
- **Exports/endpoints:** 
- **Depends on:** 
- **Known issues:** 

### `frontend/src/...`
- **Lines:** 
- **Purpose:** 
- **Exports:** 
- **Known issues:** 

---

## API surface

| Method | Path | Auth required | Request body | Response |
|--------|------|--------------|--------------|---------|
| GET | /api/... | Yes/No | — | [...] |
| POST | /api/... | Yes/No | {field: type} | {field: type} |

---

## Data models

| Model | Fields | Validations | Notes |
|-------|--------|-------------|-------|
| [ModelName] | id, name, created_at | name required, min 2 chars | |

---

## Code health snapshot

| Metric | Value | Target |
|--------|-------|--------|
| Total source files | | |
| Files > 300 lines | | 0 |
| Known TODO markers | | ≤ 5 |
| Test coverage (approx) | | ≥ 60% |
| Last audit score | N/A | ≥ 85 |

---

## Security notes

- **Auth method:** [JWT / session / API key / OAuth2 / none]
- **JWT secret:** loaded from `JWT_SECRET` env var / not applicable
- **Secrets management:** `.env` gitignored; `.env.example` with placeholders in repo
- **CORS policy:** [list allowed origins]
- **Rate limiting:** [configured at v0.X / not yet]
- **HTTPS:** [enforced in prod / pending]

---

## Environment variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | Yes | DB connection string | `postgresql://...` |
| `SECRET_KEY` | Yes | JWT or session secret | `your-secret-here` |
| `ANTHROPIC_API_KEY` | AI versions | LLM API key | `sk-ant-...` |
| `STRIPE_API_KEY` | Payment versions | Stripe key | `sk_test_...` |

---

## Dependencies

### Production (backend)
| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| | | | |

### Production (frontend)
| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| | | | |

*Flag any GPL/AGPL packages for commercial projects.*

---

## Roadmap

| Version | Status | Description | Acceptance criterion |
|---------|--------|-------------|---------------------|
| v0.0 | planned | Project skeleton | Dev server starts with no errors |
| v0.1 | planned | Core data model | CRUD endpoints return correct status codes |
| v0.2 | planned | [feature] | [criterion] |
| vX.Y | planned | Beta | All audit agents score ≥ 85 |

*Status: planned / in-progress / complete / skipped*

---

## Architecture decision log

| Date | Decision | Alternatives considered | Reason |
|------|----------|------------------------|--------|
| | | | |

---

## Known technical debt

| Item | File | Severity | Target version to fix |
|------|------|----------|----------------------|
| | | low/med/high | |

---

## Agent communication

*Updated by Claude Code and audit agents after each session.*

- **Last build session:** [date]
- **Last audit run:** [date]
- **Last audit score:** Agent1: —, Agent2: —, Agent3: —, Avg: —
- **Blockers from last audit:** [list or "none"]
- **Next session targets:** [list files or features]
