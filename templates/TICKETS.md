# Tickets — ownership ledger
*Required when more than one tool (Claude Code, OpenCode) writes to the repo.*
*Rules: one ticket = one owner = one branch `fix/<ID>-<slug>`. Writer ≠ reviewer.*
*Never edit files of a ticket you don't own. Update the row on every state change.*

| ID | Prio | Files | Fix | Acceptance | Owner | Reviewer | Branch | Status | Verdict |
|----|------|-------|-----|------------|-------|----------|--------|--------|---------|
| T-001 | P0 | `path/file.py` | [one line] | [test/command that proves it] | claude-code | opencode | fix/T-001-slug | todo | — |

*Status: todo / in-progress / in-review / merged / wontfix*
*Verdict: APPROVED / APPROVED WITH NOTES / BLOCKED*
