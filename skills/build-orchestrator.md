---
name: build-orchestrator
description: >
  Coordinates the multi-agent code generation workflow for any programming
  project. Trigger this skill when the user says "build this version",
  "implement v0.X", "write the code for", "generate the files for", "start
  coding", or "implement the plan". Also trigger when a PROJECT_MANIFEST.md
  exists and the user wants to move from planning to implementation.
  This skill manages the sequence: Claude Code generates → this chat reviews
  architecture → co-agent double-checks each new file → SESSION_HANDOFF.md
  is updated. Never generate code without first reading PROJECT_MANIFEST.md
  and SESSION_HANDOFF.md.
---

# Build Orchestrator

You coordinate the code generation phase. You do not write all the code
yourself — you define what should be built (files, contracts, interfaces)
and verify what Claude Code produces.

---

## Pre-build checklist

Before any code is generated, confirm all of the following. If any item is
missing, stop and resolve it.

- [ ] `docs/PROJECT_MANIFEST.md` exists and is complete
- [ ] `docs/SESSION_HANDOFF.md` exists and targets a specific version
- [ ] The target version's acceptance criterion is written and understood
- [ ] The tech stack is explicit (no ambiguity about language/framework versions)
- [ ] `.env.example` exists (never real secrets — see Security rules below)

---

## Build sequence (per version)

### Phase A — Specify before coding

For each file that needs to be created or modified, write a **file spec** before
any code is generated. A file spec is a brief contract:

```markdown
### File: backend/app/routers/users.py
- **Purpose:** HTTP layer for user CRUD. No business logic.
- **Exports:** router (APIRouter), mounted at /api/users
- **Depends on:** app.services.users, app.models.user, app.core.security
- **Endpoints:** GET /users, GET /users/{id}, POST /users, PUT /users/{id}, DELETE /users/{id}
- **Does NOT:** touch the database directly, contain validation logic
```

Present all file specs for the current version before Claude Code writes a
single line. The user must approve the specs.

### Phase B — Generation order

Always generate files in dependency order (bottom-up):

1. Configuration and environment (`core/config.py`, `.env.example`)
2. Data models / schema (`models/`)
3. Database layer / repositories (`services/` or `repositories/`)
4. Business logic (`services/`)
5. HTTP layer / controllers (`routers/` or `controllers/`)
6. Entry point (`main.py`, `index.ts`, `app.py`)
7. Frontend: types → hooks/services → components → pages
8. Tests (after the code they test)

Never generate a file that imports from a file that hasn't been generated yet.

### Phase C — Per-file review (the double-check loop)

After Claude Code generates each file, perform this review before moving on:

**Structural checks (always):**
- Does the file match its spec exactly?
- Are all imports resolvable (no importing non-existent modules)?
- Is there any business logic in the HTTP layer, or any HTTP concerns in the
  service layer? (This is the most common violation.)
- Are there hardcoded secrets, API keys, or credentials? → Block immediately.
- Are there TODO/FIXME comments that should block this version?

**Language-specific checks:**

*Python:*
- Pydantic models use correct field types and validators
- Async functions are awaited; sync functions don't call async
- Exception handling uses specific exception types (no bare `except:`)
- No mutable default arguments

*TypeScript/JavaScript:*
- No `any` types unless explicitly justified
- Props are typed (no untyped component props)
- Async functions have error boundaries
- No `console.log` left in production paths

*General:*
- Functions longer than 40 lines should be split (flag, don't auto-fix)
- Magic numbers have named constants
- No copy-pasted blocks (flag as future refactor target)

**Verdict per file:** `APPROVED`, `APPROVED WITH NOTES`, or `BLOCKED`.
A `BLOCKED` file must be regenerated before the build continues.

### Phase D — Integration check

After all files for a version are approved individually, verify the integration:

1. Run the project locally (or describe the exact commands to do so)
2. Verify the acceptance criterion from the roadmap is met
3. Confirm no new files were created that aren't in the manifest (stray files
   are a sign of scope creep)
4. Update `docs/PROJECT_MANIFEST.md` with any new files created

---

## Security rules (enforced at every build)

These are non-negotiable. Violation of any of these blocks the build.

1. **No secrets in source.** `.env` files are gitignored. Only `.env.example`
   with placeholder values lives in the repo.
2. **Auth tokens belong in the backend.** No API keys, JWT secrets, or service
   credentials in frontend code.
3. **CORS must be explicit.** `allow_origins=["*"]` is forbidden in non-dev
   builds. Origins must be listed.
4. **Passwords are hashed.** Any code storing or comparing passwords must use
   bcrypt, argon2, or equivalent. Never MD5, SHA1, or plaintext.
5. **SQL queries use parameterized inputs.** No string concatenation in queries.
6. **Dependencies are pinned.** `requirements.txt` uses `==`, `package.json`
   uses exact versions or `^` with lockfile committed.

---

## Updating SESSION_HANDOFF.md

At the end of every build session (whether or not the version is complete),
update `docs/SESSION_HANDOFF.md`:

- Mark completed files with their approval status
- List any blocked files and why
- Update "next session targets"
- Add any architectural decisions made during the session to
  "Architecture decision log"
- If the version's acceptance criterion is met, mark the version complete and
  set the next version as the target

---

## Handoff to the audit pipeline

Once a version's acceptance criterion is met and all files are approved, the
build phase is complete. The next step is the audit pipeline.

Instruct the user to run:
```bash
python scripts/audit_pipeline.py --manifest docs/PROJECT_MANIFEST.md
```

Or, if running audits manually, activate the audit skills in order:
1. `audit-security` (Agent 1)
2. `audit-parity` (Agent 2)
3. `audit-quality` (Agent 3)

Do not consider a version "done" until the audit pipeline produces a score ≥ 85.
