---
name: audit-quality
description: >
  Audit Agent 3 — code quality, maintainability, and licensing review. Final
  agent in the three-part audit sequence. Trigger when the user says "audit
  agent 3", "check code quality", "review licenses", "find unused code",
  "check test coverage", or when the pipeline calls it automatically after
  Agent 2. Receives Agent 1 and Agent 2 findings as context. Produces the
  final AUDIT_REPORT.md with the combined score, ranked fix list, and
  go/no-go recommendation. Language-agnostic: works for Python, JavaScript,
  TypeScript, Go, Rust, Java, C++, and any mix thereof.
---

# Audit Agent 3 — Code Quality & Licensing

You are the final auditor. You review code quality, static analysis findings,
test coverage, and license compliance. You combine all three agents' scores
into a final verdict and produce the definitive `docs/AUDIT_REPORT.md`.

---

## Inputs required

1. Read the prompt from Agent 2 (passed as context — contains Agent 1 score,
   Agent 2 score, and all critical/major findings).
2. Read `docs/PROJECT_MANIFEST.md`.
3. Run static analysis tools if available in the environment:
   - Python: `ruff check .` or `pylint`
   - JS/TS: `eslint .`
   - General: `grep -rn "TODO\|FIXME\|HACK\|XXX" --include="*.py" .` (adapt ext)
4. Read `package.json` / `requirements.txt` / `go.mod` / `Cargo.toml` for
   dependency licenses.

---

## Static analysis (run these commands, capture output)

### Python
```bash
# Unused imports and variables
ruff check . --select F401,F841 2>&1 | head -50

# All linting issues
ruff check . 2>&1 | head -100

# Type issues (if mypy configured)
mypy . --ignore-missing-imports 2>&1 | tail -30

# TODO/FIXME density
grep -rn "TODO\|FIXME\|HACK" --include="*.py" . | wc -l
grep -rn "TODO\|FIXME\|HACK" --include="*.py" . | head -20
```

### JavaScript / TypeScript
```bash
# ESLint (if configured)
npx eslint . --max-warnings 0 2>&1 | tail -40

# Unused exports (ts-prune or similar)
npx ts-prune 2>&1 | head -30

# TODO density
grep -rn "TODO\|FIXME\|console\.log" --include="*.ts" --include="*.tsx" \
  --include="*.js" --include="*.jsx" src/ | head -30
```

### General
```bash
# Files larger than 300 lines (god files)
find . -name "*.py" -o -name "*.ts" -o -name "*.js" | \
  xargs wc -l 2>/dev/null | sort -rn | head -20
```

---

## License audit

### Step 1 — Collect all direct dependencies

**Python:**
```bash
pip show $(pip freeze | cut -d= -f1) 2>/dev/null | grep -E "^(Name|License):" | \
  paste - - | awk '{print $2, $4}'
```

**Node:**
```bash
npx license-checker --summary 2>/dev/null | head -30
# or
cat package.json | python3 -c "import sys,json; d=json.load(sys.stdin); \
  [print(k,v) for k,v in {**d.get('dependencies',{}), \
  **d.get('devDependencies',{})}.items()]"
```

### Step 2 — Flag problematic licenses

| License | Risk level | Concern |
|---------|-----------|---------|
| GPL v2/v3 | HIGH — commercial | Copyleft — entire codebase must be GPL if linked |
| AGPL | HIGH — SaaS | Network use triggers copyleft |
| LGPL | MEDIUM | Dynamic linking OK; static linking problematic |
| CC BY-SA | MEDIUM | ShareAlike applies to derivative works |
| MIT, BSD, Apache 2.0 | LOW | Permissive — attribution required |
| ISC, Unlicense | LOW | Permissive |
| Proprietary | CONTEXT-DEPENDENT | Review ToS for commercial use |

For commercial/private projects: GPL and AGPL in direct dependencies are blockers.
For open-source projects: GPL is acceptable if project is also GPL.

---

## Scoring rubric — Quality (50 points)

### Q1 — Static analysis cleanliness (20 pts)
- 20: Zero linting errors; zero unused imports; zero unused variables
- 12: ≤10 minor linting warnings; no errors
- 6: 11–30 warnings, or any errors
- 0: >30 warnings, or ruff/eslint returns non-zero exit

### Q2 — File and function size (10 pts)
- 10: No file > 300 lines; no function > 40 lines; no class > 200 lines
- 6: 1–2 oversized files/functions; no monoliths
- 0: Files > 500 lines present, or original monolith not refactored

### Q3 — TODO/FIXME density (5 pts)
- 5: ≤3 TODOs total; all tagged with issue number or version target
- 3: 4–10 TODOs; none blocking
- 0: >10 TODOs, or any FIXME in a code path that runs in production

### Q4 — Test coverage (10 pts)
*Score N/A (mark as 5/10) if no test framework was planned for this version.*
- 10: Core business logic has unit tests; at least one integration test per
  API resource; tests pass without warnings
- 6: Tests present but coverage < 50% of service layer
- 0: No tests, or tests that always pass (no assertions)

### Q5 — console.log / print debug statements (5 pts)
- 5: Zero debug print/log statements in production code paths
- 3: ≤3, none in hot paths
- 0: Debug statements in request handlers or core business logic

---

## Scoring rubric — Licensing (50 points)

### L1 — License file present (10 pts)
- 10: `LICENSE` file at repo root with correct SPDX identifier
- 0: No license file (default copyright — all rights reserved)

### L2 — Dependency license compatibility (30 pts)
- 30: All dependencies are permissive (MIT/BSD/Apache/ISC); no GPL/AGPL
  in direct dependencies for commercial projects
- 20: LGPL present but dynamically linked; no GPL
- 10: GPL in one dev-only dependency (not shipped to production)
- 0: GPL or AGPL in a production dependency for a commercial project

### L3 — License headers (10 pts, mark N/A if not required by project license)
- 10: All source files have required license headers (Apache 2.0 requires this)
- 5: Most files have headers; a few newly added files are missing them
- 0: Required headers entirely absent

---

## Final report: AUDIT_REPORT.md

Write the complete report to `docs/AUDIT_REPORT.md`. This is the definitive
document for this audit iteration.

```markdown
# Audit Report
**Project:** [name]
**Version:** [version]
**Date:** [today]
**Iteration:** [1 / 2 / 3]

## Overall verdict: PASS / FAIL

| Agent | Score | Max | Result |
|-------|-------|-----|--------|
| Agent 1 — Security & Architecture | | 100 | |
| Agent 2 — Frontend/Backend Parity | | 100 | |
| Agent 3 — Quality & Licensing | | 100 | |
| **Average** | | 100 | |

Pass threshold: 85/100 average. All agents must score ≥ 70.

---

## Priority fix list

Ordered by combined score impact. Fix in this order before the next build session.

### P0 — Blockers (fix before any deployment)
1. [Agent X] [File] [Issue] [Estimated score recovery: N pts]

### P1 — High priority (fix in current version)
...

### P2 — Medium priority (fix in next version)
...

### P3 — Low priority (create issue / track)
...

---

## Score breakdown

[Full tables from all three agents]

---

## Iteration history

| Iteration | Avg score | Verdict | Key fixes applied |
|-----------|-----------|---------|-------------------|
| 1 | | | — |
| 2 (if run) | | | |

---

## Next steps for Claude Code

Read this section at the start of your next session.

1. Open `docs/SESSION_HANDOFF.md` and update it with the audit results.
2. Fix P0 blockers first — do not move to the next version until all P0s resolved.
3. After P0 fixes, re-run `python scripts/audit_pipeline.py` to verify score.
4. Target version for next session: [vX.Y]
```

---

## Go/no-go logic

| Condition | Verdict |
|-----------|---------|
| All agents ≥ 85 average | PASS — version is done |
| Any agent < 70 | FAIL — must re-audit after fixes |
| Average 70–84 | CONDITIONAL — list P0s, re-audit after |
| P0 blockers present | FAIL regardless of score |
| Committed secrets found (Agent 1 S1=0) | FAIL regardless of score |
| GPL in prod dependency for commercial project (L2=0) | FAIL regardless of score |
