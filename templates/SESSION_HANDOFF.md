# Session Handoff
*Updated at the end of every build session by the build orchestrator.*
*Read this at the START of every new Claude Code session before touching any code.*

---

## Current state

**Project:** [name]
**Date of last session:** [YYYY-MM-DD HH:MM]
**Last completed version:** [vX.Y]
**Version acceptance status:** [met / not yet met / partially met]
**Next target version:** [vX.Y]
**Next version acceptance criterion:**
> [Copy the acceptance criterion from the roadmap verbatim]

---

## What was done in the last session

*List every file created, modified, or deleted.*

| File | Action | Status | Notes |
|------|--------|--------|-------|
| `backend/app/routers/users.py` | created | APPROVED | |
| `frontend/src/services/api.ts` | modified | APPROVED WITH NOTES | See note below |
| `backend/app/models/user.py` | created | BLOCKED | Missing validator |

**Notes on approved-with-notes files:**
- [file]: [what to watch for]

**Reason for any BLOCKED files:**
- [file]: [precise reason] → [what must be done to unblock]

---

## Open blockers

*Must be resolved before this version is considered done.*

1. [Blocker description] — [file or component affected] — [suggested fix]

---

## Last audit results

| Agent | Score | Key findings |
|-------|-------|-------------|
| Agent 1 — Security | —/100 | |
| Agent 2 — Parity | —/100 | |
| Agent 3 — Quality | —/100 | |
| Average | —/100 | |

**Audit verdict:** [PASS / FAIL / CONDITIONAL / not yet run]

**P0 blockers from last audit:**
- [item] — [file] — [fix required]

---

## Architecture decisions made this session

*Anything that affects future sessions — record it here AND in PROJECT_MANIFEST.md.*

- [Decision]: [Reason]

---

## Things to watch for next session

*Warnings, fragile areas, or unfinished threads.*

- [Item]

---

## Next session task list

*Ordered by priority. Do not reorder.*

1. Fix P0 audit blockers (if any):
   - [ ] [item]
2. Implement [feature/file] for v[X.Y]:
   - [ ] [file spec]
   - [ ] [file spec]
3. Run audit pipeline when v[X.Y] acceptance criterion is met:
   ```bash
   python scripts/audit_pipeline.py --manifest docs/PROJECT_MANIFEST.md
   ```
4. Update PROJECT_MANIFEST.md with new files
5. Update this SESSION_HANDOFF.md

---

## Environment setup reminder

```bash
# Install backend dependencies
pip install -r requirements.txt

# Install frontend dependencies
npm install

# Set environment variables
cp .env.example .env
# Edit .env with real values

# Start dev servers
[your start command]
```

---

## Context for co-agents

*If another AI tool (ChatGPT, OpenCode Go, etc.) needs to review this session's work,
paste this section into their context window.*

**Project:** [name] — [one-sentence description]
**Stack:** [tech stack]
**Version being built:** [vX.Y]
**Files changed this session:**
[list]
**Key concern for reviewer:** [what to focus on]
