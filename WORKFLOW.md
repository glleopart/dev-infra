# Dev-Infra Workflow Reference
*Save this in ~/.dev-infra/ — your complete methodology reference.*

---

## The two tools and what each does

| Tool | Purpose |
|---|---|
| **Claude.ai chat** | Plan, review, learn. No code written here. |
| **Terminal (Claude Code / OpenCode)** | Agents build the actual code. |
| **`docs/SESSION_HANDOFF.md`** | The bridge between both tools. Always kept in sync. |

---

## ONE-TIME SETUP PER PROJECT

### 1. Duplicate the Claude.ai template project (browser)
- Open your `_template` Claude.ai project
- Project Settings → Duplicate → rename to your project name
- The 6 skills are already in Knowledge

### 2. Bootstrap the project (terminal)
```zsh
cd ~/projects/personal        # or ~/projects/Pymetra
new-project myapp
cd myapp
```

### 3. Planning conversation (Claude.ai — new chat)
Type: *"I want to build [description]. Let's use the project-architect skill."*

Claude runs intake, designs roadmap. At the end it produces the full
content for PROJECT_MANIFEST.md and SESSION_HANDOFF.md.

**→ Copy each block → paste into the files on disk:**
```zsh
nano docs/PROJECT_MANIFEST.md    # paste, save
nano docs/SESSION_HANDOFF.md     # paste, save
```

### 4. Upload project docs to Claude.ai Knowledge
Project Settings → Knowledge → upload:
- `docs/PROJECT_MANIFEST.md`
- `docs/SESSION_HANDOFF.md`

These are now auto-injected into every future chat in this project.

---

## FOR EVERY VERSION (v0.0 → beta)

### PHASE A — PLAN (Claude.ai, new chat)

Open new chat. Type: *"Let's plan v0.X"*

Claude reads SESSION_HANDOFF.md from Knowledge automatically,
designs file specs in dependency order. You approve.

Claude outputs a **session task block** at the end:
```
SESSION TASK FOR CLAUDE CODE:
Read SESSION_HANDOFF.md. Build v0.X.
Files to create in order: [list with specs]
Acceptance criterion: [criterion]
```

**→ Copy this block → paste into the "Next session targets"
   section of docs/SESSION_HANDOFF.md on disk → save**

```zsh
nano docs/SESSION_HANDOFF.md
```

**→ Re-upload SESSION_HANDOFF.md to Claude.ai Knowledge**
   (replace the old version)

→ Close or minimise Claude.ai.

---

### PHASE B — BUILD (terminal)

```zsh
cd ~/projects/personal/myapp
```

**Claude Code:**
```zsh
claude
# First message: "Start the session. Read SESSION_HANDOFF.md and build v0.X."
# When done: /exit or Ctrl+C
```

**OpenCode:**
```zsh
opencode
# First message: "@orchestrator Start the session. Read SESSION_HANDOFF.md and build v0.X."
# When done: Ctrl+C or exit
```

Orchestrator reads SESSION_HANDOFF.md → delegates to builder
one file at a time → each file reviewed → you watch and ask questions.

When version is done, orchestrator updates SESSION_HANDOFF.md on disk.

---

### PHASE C — AUDIT (terminal)

```zsh
audit                    # runs 3 agents, writes docs/AUDIT_REPORT.md
audit --score            # check result
```

**Score ≥ 85 → PASS → go to Phase D**

**Score < 85 → FAIL:**
```zsh
cat docs/AUDIT_REPORT.md       # read P0 blockers
claude                         # reopen Claude Code
# "Fix the P0 blockers from docs/AUDIT_REPORT.md"
audit                          # re-run after fixes
```
Repeat until PASS.

---

### PHASE D — REVIEW + LEARN (Claude.ai, new chat)

```zsh
cat docs/AUDIT_REPORT.md       # copy all output
```

Open new chat in project. Type:
*"v0.X passed. Here's the audit report: [paste content]"*

Claude explains findings, teaches patterns, runs tutor debrief.
Ask everything you don't understand.

**→ Re-upload SESSION_HANDOFF.md to Knowledge** (orchestrator
   updated it during the build — Knowledge version is now stale)

**→ Re-upload PROJECT_MANIFEST.md if it changed** (new files
   added, API surface changed):
```zsh
python3 scripts/generate_manifest.py --update
# Then re-upload docs/PROJECT_MANIFEST.md to Knowledge
```

→ Version complete. Start Phase A again for the next version.

---

## File movement reference

| File | Updated by | Disk → Claude.ai | Claude Code reads |
|---|---|---|---|
| `SESSION_HANDOFF.md` | You (planning) + orchestrator (build) | Re-upload to Knowledge after every update | From disk |
| `PROJECT_MANIFEST.md` | `new-project` + `generate_manifest.py` | Re-upload when changed | From disk |
| `AUDIT_REPORT.md` | `audit` command | Copy-paste into chat | Written to disk |
| Session task block | Claude.ai planning chat | Paste into SESSION_HANDOFF.md on disk | From disk |

---

## The one rule

**SESSION_HANDOFF.md is always the bridge.**
- After planning → you update it on disk → re-upload to Knowledge
- After building → orchestrator updates it on disk → you re-upload to Knowledge
- It is the single source of truth about where the project is at all times

---

## Quick reference card

```
NEW PROJECT:
  browser  → duplicate template → new chat → plan → copy to docs/ → upload to Knowledge
  terminal → new-project name

EACH VERSION:
  browser  → new chat → "plan v0.X" → copy session task → paste into SESSION_HANDOFF.md
           → re-upload SESSION_HANDOFF.md
  terminal → claude / opencode → "start session, build v0.X" → exit when done
  terminal → audit → fix P0s if needed → re-audit until PASS
  browser  → new chat → paste AUDIT_REPORT.md → learn → re-upload SESSION_HANDOFF.md
```

---

## Agent invocation reference

| Agent | Claude Code | OpenCode |
|---|---|---|
| orchestrator | automatic on session start | `@orchestrator` |
| builder | delegated by orchestrator | `@builder` |
| tutor | `/agents → tutor` | `@tutor` |
| audit agents | via `audit` command | via `audit` command |

---

## Skill trigger reference

| Skill file | Activates when you say |
|---|---|
| `project-architect.md` | "plan my project", "I want to build X" |
| `build-orchestrator.md` | "build v0.X", "implement", "write the code" |
| `audit-security.md` | "security review", "audit agent 1" |
| `audit-parity.md` | "parity review", "audit agent 2" |
| `audit-quality.md` | "quality review", "check licenses", "audit agent 3" |
| `ml-integration.md` | "add AI", "LLM", "chatbot", "recommender" |
