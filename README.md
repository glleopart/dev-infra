# ~/.dev-infra — Personal Dev Infrastructure

A language-agnostic, multi-agent framework for planning, building, and
auditing any software project from scratch to beta.

Works with **Claude Code**, **OpenCode**, and **Codex CLI**.

---

## Folder structure

```
~/.dev-infra/
├── agents/                        # Autonomous terminal agents (Claude Code / OpenCode)
│   ├── orchestrator.md            # Plans versions, writes specs, delegates — never codes
│   ├── builder.md                 # Implements one file at a time from a spec
│   ├── security-auditor.md        # Audit Agent 1: security & architecture (scores /100)
│   ├── parity-auditor.md          # Audit Agent 2: frontend/backend consistency (scores /100)
│   ├── quality-auditor.md         # Audit Agent 3: quality & licensing → writes AUDIT_REPORT.md
│   └── tutor.md                   # Explains what was built; maximises learning per session
│
├── bin/                           # Global CLI commands (on $PATH via .zshrc)
│   ├── new-project                # Bootstrap any new project with the full infrastructure
│   ├── audit                      # Run the 3-agent audit pipeline from anywhere in a project
│   └── install-agents             # Install agents globally into Claude Code and OpenCode
│
├── scripts/                       # Python scripts copied into each project
│   ├── audit_pipeline.py          # Orchestrates the 3-agent audit via Anthropic API
│   └── generate_manifest.py       # Auto-generates docs/PROJECT_MANIFEST.md from any codebase
│
├── skills/                        # Claude.ai project knowledge files (flat .md, no subfolders)
│   ├── project-architect.md       # Session 0: intake, roadmap, manifest generation
│   ├── build-orchestrator.md      # Build phase: file specs, per-file review, handoff
│   ├── audit-security.md          # Audit Agent 1 instructions (used in claude.ai chat)
│   ├── audit-parity.md            # Audit Agent 2 instructions
│   ├── audit-quality.md           # Audit Agent 3 instructions
│   └── ml-integration.md          # ML/DL/AI layer patterns and integration guide
│
├── templates/                     # Copied into docs/ of every new project
│   ├── PROJECT_MANIFEST.md        # Central project map — every agent reads this
│   ├── SESSION_HANDOFF.md         # State passed between sessions and agents
│   ├── AUDIT_PROMPTS.md           # Per-project audit customisation and thresholds
│   └── VERSION_PLAN.md            # Detailed breakdown template for one version
│
├── README.md                      # This file
└── SETUP.md                       # One-time machine setup instructions
```

---

## Two types of AI artefacts — and why both exist

| | Skills (`skills/*.md`) | Agents (`agents/*.md`) |
|---|---|---|
| **Where used** | Claude.ai project chat | Claude Code / OpenCode terminal |
| **What they do** | Guide Claude's conversational behaviour | Run autonomously with tools, model choice, and permissions |
| **When active** | During planning and design conversations | During build and audit sessions in the terminal |
| **Format** | YAML frontmatter + instructions | YAML frontmatter + system prompt |

Skills and agents are complementary. You plan in Claude.ai (skills active),
then build and audit in the terminal (agents active).

---

## Full workflow

```
SESSION 0 — PLAN (claude.ai, skills active)
  Describe your project → project-architect skill activates
  Together: roadmap, folder structure, AGENTS.md, PROJECT_MANIFEST.md
  Output: approved versioned plan (v0.0 → beta)

  $ new-project myapp          # bootstrap the project in the terminal

FOR EACH VERSION:

  STEP A — PLAN THE VERSION (claude.ai or terminal plan mode)
    orchestrator reads SESSION_HANDOFF.md
    writes file specs → you approve

  STEP B — BUILD (terminal: Claude Code / OpenCode / Codex)
    orchestrator delegates → builder implements one file at a time
    each file reviewed before moving to the next

  STEP C — AUDIT (terminal)
    $ audit
    → security-auditor (Agent 1) → parity-auditor (Agent 2) → quality-auditor (Agent 3)
    → docs/AUDIT_REPORT.md written
    → project rules from docs/AUDIT_PROMPTS.md are injected into every agent
    → fix P0/P1, re-run until avg ≥ 85

  STEP D — LEARN (claude.ai)
    tutor explains patterns, code decisions, concepts introduced
    flash-card Q&A, one experiment to try yourself

  STEP E — TEST (you)
    manual testing → back to STEP B if fixes needed
```

---

## One-time machine setup

See `SETUP.md` for the full walkthrough. The short version:

```zsh
# 1. Place this folder
mv ~/Downloads/dev-infrastructure ~/.dev-infra

# 2. Make bin scripts executable
chmod +x ~/.dev-infra/bin/new-project \
         ~/.dev-infra/bin/audit \
         ~/.dev-infra/bin/install-agents

# 3. Add to ~/.zshrc (API key only for `audit --api`; keep it in ~/.secrets)
export DEV_INFRA_DIR="$HOME/.dev-infra"
export PATH="$DEV_INFRA_DIR/bin:$PATH"
[[ -f "$HOME/.secrets" ]] && source "$HOME/.secrets"

# 4. Reload and install Python deps
source ~/.zshrc
pip install anthropic tenacity

# 5. Install agents globally into your coding tools
install-agents

# 6. Upload skills to each new Claude.ai project
#    (Project Settings → Knowledge → upload the 6 files in skills/)
```

---

## Starting a new project

```zsh
cd ~/projects
new-project my-app-name
# Follow the printed instructions to open Claude.ai and start planning
```

---

## Running an audit

```zsh
# From anywhere inside a project
audit                   # single pass via `claude` CLI, auto-detects root
audit --api             # use the Anthropic API instead (needs key)
audit --score           # print last result, no API calls
audit --dry-run         # list files that would be sent (+ omitted ones)
audit --threshold 90    # stricter pass bar (default: 85)
audit --max-iter 2      # pause for fixes, then re-audit fresh code
audit --max-files 200   # raise the file budget for large repos
```

---

## Agent quick reference

| Agent | Model | Invoke in Claude Code | Invoke in OpenCode |
|-------|-------|-----------------------|--------------------|
| orchestrator | opus | main session (via CLAUDE.md) | `@orchestrator` (primary) |
| builder | Sonnet | Delegated by orchestrator | `@builder` |
| security-auditor | Sonnet | Via `audit` command | Via `audit` command |
| parity-auditor | Sonnet | Via `audit` command | Via `audit` command |
| quality-auditor | Sonnet | Via `audit` command | Via `audit` command |
| tutor | Opus | `/agents → tutor` | `@tutor` |

---

## Skills quick reference

Upload these to every new Claude.ai project (Project Settings → Knowledge):

| File | Activates when you say |
|------|------------------------|
| `project-architect.md` | "plan my project", "design the architecture", "I want to build X" |
| `build-orchestrator.md` | "build v0.X", "implement", "write the code for" |
| `audit-security.md` | "security review", "audit agent 1", "check vulnerabilities" |
| `audit-parity.md` | "parity review", "audit agent 2", "frontend backend mismatch" |
| `audit-quality.md` | "quality review", "audit agent 3", "check licenses" |
| `ml-integration.md` | "add AI", "integrate LLM", "chatbot", "recommender" |

---

## Audit score thresholds

| Score | Meaning |
|-------|---------|
| ≥ 85 avg, each agent ≥ 70 | **PASS** — version is done |
| 70–84 avg | **CONDITIONAL** — fix P0s then re-audit |
| Any agent < 70 | **FAIL** — critical issues |
| Agent 1 S1 = 0 | **AUTO-FAIL** — committed secret found |
| Agent 3 L2 = 0 | **AUTO-FAIL** — GPL in commercial production dependency |

---

## Updating the infrastructure

Edit source files in `~/.dev-infra/`, then:

```zsh
install-agents          # re-sync agents to Claude Code and OpenCode
# Re-upload changed skill files to any affected Claude.ai projects
```

---

## Requirements

- Python 3.9+ in a pyenv/conda env: `pip install anthropic tenacity pip-licenses`
- `claude` CLI (default audit mode) or `ANTHROPIC_API_KEY` in ~/.secrets (`--api`)
- Claude Code and/or OpenCode installed

---

## License

MIT — use in any project, commercial or otherwise.
