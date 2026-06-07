# Dev Infrastructure — One-Time Setup Guide

Run these steps once on your machine. After that, every new project is a
single command.

---

## Step 1 — Place the infrastructure folder

Move the downloaded `dev-infrastructure/` folder to your home directory and
rename it `.dev-infra`:

```zsh
mv ~/Downloads/dev-infrastructure ~/.dev-infra
```

Your final structure should be:

```
~/.dev-infra/
├── bin/
│   ├── new-project        ← creates new projects
│   └── audit              ← runs the audit pipeline
├── scripts/
│   ├── audit_pipeline.py
│   └── generate_manifest.py
├── skills/
│   ├── project-architect/SKILL.md
│   ├── build-orchestrator/SKILL.md
│   ├── audit-security/SKILL.md
│   ├── audit-parity/SKILL.md
│   ├── audit-quality/SKILL.md
│   └── ml-integration/SKILL.md
└── templates/
    ├── PROJECT_MANIFEST.md
    ├── SESSION_HANDOFF.md
    ├── AUDIT_PROMPTS.md
    └── VERSION_PLAN.md
```

---

## Step 2 — Make the bin scripts executable

```zsh
chmod +x ~/.dev-infra/bin/new-project
chmod +x ~/.dev-infra/bin/audit
```

---

## Step 3 — Add to PATH in .zshrc

Open your `~/.zshrc` in any editor and add these lines at the bottom:

```zsh
# ── Dev infrastructure ─────────────────────────────────────────────
export DEV_INFRA_DIR="$HOME/.dev-infra"
export PATH="$DEV_INFRA_DIR/bin:$PATH"

# Anthropic API key (used by the audit pipeline)
# Option A: hardcode here (simplest)
export ANTHROPIC_API_KEY="sk-ant-your-key-here"

# Option B: load from a secrets file (more secure — add ~/.secrets to .gitignore)
# source "$HOME/.secrets"
```

Then reload your shell:

```zsh
source ~/.zshrc
```

Verify it works:

```zsh
which new-project    # should print: /Users/you/.dev-infra/bin/new-project
which audit          # should print: /Users/you/.dev-infra/bin/audit
new-project --help
```

---

## Step 4 — Install Python dependencies (once)

The audit pipeline requires two packages:

```zsh
pip install anthropic tenacity
```

If you use multiple Python environments, install in whichever Python is your
default `python3`. Check with: `which python3`.

---

## Step 5 — Install skills in Claude.ai (once per Claude project)

Skills are uploaded per Claude project, not globally. Each time you create a
new Claude project for a new codebase:

1. Go to [claude.ai](https://claude.ai) → open or create a Project
2. Click the project name → **Project Settings**
3. Find the **Knowledge** or **Project files** section
4. Upload these 6 files (drag and drop, or click Add):

```
~/.dev-infra/skills/project-architect/SKILL.md
~/.dev-infra/skills/build-orchestrator/SKILL.md
~/.dev-infra/skills/audit-security/SKILL.md
~/.dev-infra/skills/audit-parity/SKILL.md
~/.dev-infra/skills/audit-quality/SKILL.md
~/.dev-infra/skills/ml-integration/SKILL.md
```

You only need `ml-integration` if the project will use AI. The other 5 are
always useful.

> Tip: keep one "template Claude project" with all 6 skills already uploaded.
> Duplicate it for each new project via Project Settings → Duplicate.

---

## Daily usage: starting a new project

```zsh
# Navigate to where you keep your projects
cd ~/projects

# Bootstrap a new project
new-project my-app-name

# The script will:
# 1. Create ~/projects/my-app-name/
# 2. Copy scripts/ and templates/ into it
# 3. Generate docs/PROJECT_MANIFEST.md
# 4. Init git
# 5. Print the first message to paste into Claude

# Open Claude, paste the printed message, and start building
```

For an existing repo:

```zsh
new-project existing-repo --from ~/projects/existing-repo
```

---

## Daily usage: running audits

```zsh
# From anywhere inside a project (any subdirectory works)
cd ~/projects/my-app-name/backend
audit

# Shortcuts
audit --score          # print last score, no new API calls
audit --dry-run        # see which files would be sent (no API calls)
audit --threshold 90   # stricter bar (default: 85)
audit --max-iter 1     # single pass, no auto-retry
```

The script auto-finds the project root by walking up from your current
directory. You never need to specify paths manually.

---

## Updating the infrastructure

When you improve the skills, scripts, or templates, update them in `~/.dev-infra/`.
Changes take effect immediately for new projects. To apply to an existing project:

```zsh
# Re-copy scripts (safe — doesn't touch docs/ or your code)
cp ~/.dev-infra/scripts/*.py ~/projects/my-app/scripts/

# Skills: re-upload to the Claude project (replace the old files)
```

---

## Optional: zshrc aliases

Add these to `~/.zshrc` for even faster access:

```zsh
# Quick audit shortcuts
alias a="audit"
alias adry="audit --dry-run"
alias ascore="audit --score"

# Open the infra folder
alias infra="cd $DEV_INFRA_DIR"

# Open the manifest for the current project
alias manifest="open docs/PROJECT_MANIFEST.md 2>/dev/null || cat docs/PROJECT_MANIFEST.md"
```

---

## Quick reference card

| Command | What it does |
|---------|-------------|
| `new-project myapp` | Bootstrap a new project |
| `new-project myapp --no-git` | Bootstrap without git init |
| `new-project myapp --from /path` | Add infra to existing repo |
| `audit` | Full 3-agent audit (auto-detects project root) |
| `audit --score` | Print last audit result, no API calls |
| `audit --dry-run` | List files that would be sent |
| `audit --threshold 90` | Custom pass score |
| `audit --max-iter 1` | Single pass, no retry |

| Claude trigger | Skill that activates |
|----------------|---------------------|
| "plan my project / design the architecture" | `project-architect` |
| "build v0.X / implement / write the code" | `build-orchestrator` |
| "audit / security review / audit agent 1" | `audit-security` |
| "parity review / audit agent 2" | `audit-parity` |
| "quality review / audit agent 3 / licenses" | `audit-quality` |
| "add AI / integrate LLM / chatbot / recommender" | `ml-integration` |

---

## Troubleshooting

**`command not found: new-project`**
→ Run `source ~/.zshrc` or open a new terminal tab.
→ Check `echo $PATH` contains `~/.dev-infra/bin`.

**`ERROR: ANTHROPIC_API_KEY is not set`**
→ Add `export ANTHROPIC_API_KEY="sk-ant-..."` to `~/.zshrc` and `source ~/.zshrc`.

**`Missing Python dependency: anthropic`**
→ Run `pip install anthropic tenacity`.
→ If you use conda or pyenv, activate the right environment first.

**Audit sends wrong files / too many files**
→ Run `audit --dry-run` to see exactly what's included.
→ Add patterns to the `SKIP_DIRS` set in `scripts/audit_pipeline.py`.

**Skills not activating in Claude**
→ Make sure all 6 SKILL.md files are uploaded to the Claude project's Knowledge section.
→ The trigger description in each SKILL.md is what Claude reads — if it's not activating,
  try phrasing your request closer to the trigger phrases listed in this guide.
