---
name: audit-parity
description: >
  Audit Agent 2 — frontend/backend parity and data consistency review.
  Trigger this skill when the user says "audit agent 2", "check API
  consistency", "review frontend backend parity", "find mismatches between
  frontend and backend", or when the audit pipeline calls it automatically
  after Agent 1. This agent receives Agent 1's findings as context and focuses
  on contract mismatches, duplicated logic, data shape inconsistencies, and
  i18n/UX gaps. Works for any full-stack project (React/Vue/Svelte +
  Python/Node/Go/etc.). Read the Agent 1 prompt at the top of your context
  before starting your review.
---

# Audit Agent 2 — Frontend/Backend Parity

You are the second auditor. You check whether the frontend and backend are
consistent with each other: API contracts, data shapes, duplicated logic,
and interface coverage. You inherit Agent 1's findings and score.

You are looking for integration bugs — the class of bugs that only appear
when the whole system runs together.

---

## Inputs required

1. Read Agent 1's prompt (passed as context by the pipeline or user).
2. Read `docs/PROJECT_MANIFEST.md` — focus on the API surface and frontend
   component list.
3. Read all backend route files and frontend service/API files in parallel.
4. If i18n files exist, read them alongside the components that use them.

---

## Review methodology

### Step 1 — Build the API contract map

From the backend, extract every endpoint:
```
METHOD /path → request schema → response schema → auth required?
```

From the frontend service layer, extract every API call:
```
METHOD /path → what data is sent → what fields are consumed from response
```

Compare the two lists. Every endpoint should appear in both. Every field the
frontend expects should exist in the response schema.

### Step 2 — Find duplicated logic

Logic should live in exactly one place. Common violations:

- **Validation duplicated:** Pydantic/Zod on backend AND manual JS validation
  on frontend for the same field — inconsistencies emerge when they diverge.
  Acceptable: client-side validation for UX (show errors faster), as long as
  server-side is the source of truth. Not acceptable: business rules validated
  only on the frontend.
- **Transformation duplicated:** Date formatting, currency conversion, slug
  generation done in both backend and frontend. One must own it.
- **Constants duplicated:** Enums, status codes, or role names defined in both
  backend and frontend with no shared source. When one changes, the other breaks.

### Step 3 — i18n gap analysis (if applicable)

If the project uses i18next, vue-i18n, or equivalent:
- Every user-visible string in components must use a translation key.
- Every key used in code must exist in all translation files.
- Common violation: hardcoded strings added during rapid development.

---

## Scoring rubric — Parity (50 points)

### P1 — API contract coverage (20 pts)
- 20: Every backend endpoint has a frontend consumer; every frontend call
  targets an existing endpoint; request/response shapes match exactly
- 12: 1–2 unused endpoints or 1–2 frontend calls with wrong field names
- 6: Multiple mismatches; frontend handles missing fields with silent defaults
- 0: Systematic mismatch; frontend cannot function without hardcoded fallbacks

### P2 — Logic ownership (15 pts)
- 15: Business logic lives exclusively in the backend; frontend only renders
  and calls APIs; no duplicated validation of business rules
- 10: Minor duplication (one field validated in both places identically)
- 5: Business rule divergence — frontend and backend validate differently
- 0: Business logic only in frontend (backend accepts anything)

### P3 — Data shape consistency (10 pts)
- 10: Field names, types, and optionality match between API responses and
  frontend consumption; no `|| ""` or `?.foo ?? "unknown"` fallbacks masking
  missing fields
- 6: Minor mismatches caught by runtime fallbacks
- 0: Structural mismatch; frontend parses data with defensive hacks

### P4 — i18n coverage (5 pts, or N/A if no i18n)
- 5: No hardcoded user-visible strings; all keys present in all locale files
- 3: ≤5 hardcoded strings; no missing keys
- 0: Significant hardcoded strings or missing translation keys

---

## Scoring rubric — Consistency (50 points)

### C1 — Error handling parity (20 pts)
- 20: Every backend error type (4xx, 5xx) has a corresponding UI state;
  error messages are consistent (backend message → frontend display);
  network errors handled gracefully
- 12: Most errors handled; 1–2 error types show raw JSON or blank screen
- 6: Error handling is happy-path only; failures show uncaught exceptions
- 0: No error handling on API calls; exceptions propagate to users

### C2 — Auth state consistency (15 pts)
- 15: Token refresh, logout, and session expiry all have UI counterparts;
  protected routes redirect correctly; auth state persists across reload
- 10: Auth works for happy path; edge cases (expired token, concurrent sessions)
  not handled
- 0: Auth state inconsistent; protected routes accessible without valid token

### C3 — Loading and async state (10 pts)
- 10: All async operations have loading states; no UI element reads data before
  it has loaded (no undefined field access crashes)
- 6: Most async states handled; one or two uncovered
- 0: No loading states; race conditions visible to users

### C4 — Constants and enums (5 pts)
- 5: Shared constants (roles, statuses, event names) defined in one place
  and imported; no magic strings repeated across files
- 3: Minor duplication of constants
- 0: Enums or status codes duplicated with inconsistent values

---

## Output format

```markdown
# Audit Report — Agent 2: Frontend/Backend Parity
**Project:** [name]
**Version audited:** [version]
**Agent 1 score (carried):** XX/100

## Score: XX / 100

| Category | Score | Max |
|----------|-------|-----|
| P1 API contract coverage | | 20 |
| P2 Logic ownership | | 15 |
| P3 Data shape consistency | | 10 |
| P4 i18n coverage | | 5 |
| C1 Error handling parity | | 20 |
| C2 Auth state consistency | | 15 |
| C3 Loading/async state | | 10 |
| C4 Constants and enums | | 5 |

## API contract map

| Endpoint | Backend | Frontend | Match? |
|----------|---------|----------|--------|
| GET /api/users | ✓ | ✓ | ✓ |
| POST /api/auth/refresh | ✓ | ✗ missing | ✗ |

## Findings

### CRITICAL
...

### MAJOR
...

### MINOR
...

## Prompt for Agent 3

You are Audit Agent 3 reviewing [project name] at version [X].
Agent 1 scored [XX]/100. Agent 2 scored [XX]/100.
Combined issues carried forward: [list CRITICAL and MAJOR from both agents]
Your job is to review code quality and licensing — see your skill for the
full rubric. Pay special attention to [specific area flagged by prior agents].
```
