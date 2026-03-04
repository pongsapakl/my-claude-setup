---
name: init
description: Bootstrap workspace with docs structure and WORK.md. Auto-invokes on "/init", "initialize workspace", "set up project structure".
allowed-tools: [Bash, Write, Read, Glob, AskUserQuestion]
---

# Init — Workspace Bootstrapper

Sets up the documentation structure and live state file for a new project.

## When to Auto-Invoke

Trigger when user says:
- `/init`
- "initialize workspace"
- "set up project structure"
- "set up docs structure"
- "bootstrap project"

## How It Works

### Step 1: Detect Existing Structure

Check what already exists in the project root:

```bash
# Check for existing docs folders
ls -d docs/sessions docs/decisions docs/research docs/plans 2>/dev/null
# Check for WORK.md
ls WORK.md 2>/dev/null
# Check for legacy .claude/memory/session-logs
ls -d .claude/memory/session-logs 2>/dev/null
```

Record which items exist and which need to be created.

### Step 2: Smart Merge (if anything exists)

For **each** existing item, use AskUserQuestion:

- "docs/sessions/ already exists. What should I do?"
  - **Keep** (leave as-is)
  - **Replace** (delete and recreate empty)
  - **Skip** (don't touch)

- "WORK.md already exists. What should I do?"
  - **Keep** (leave as-is)
  - **Replace** (overwrite with fresh template)
  - **Skip** (don't touch)

If nothing exists, skip this step entirely — just create everything.

### Step 3: Create Folder Structure

For each folder the user approved (or that doesn't exist yet):

```bash
mkdir -p docs/sessions docs/decisions docs/research docs/plans
```

### Step 4: Create WORK.md

Only if it doesn't exist or user chose "Replace".

Use this template:

```markdown
# WORK.md

> Last updated: {YYYY-MM-DD}

## Current Phase
{Describe what phase the project is in}

## Just Done
- {What was accomplished in the last session}

## Blocked / Open Questions
- {Any blockers or questions that need answers}

## Next Steps
1. {What should happen next — first item matches "Next Session Starts With" from last session log}

## Active Files
- {Key files currently being worked on}

## Active Plan
None
```

Replace `{YYYY-MM-DD}` with today's date. Leave placeholder text for user to fill in.

### Step 5: Configure .gitignore

For **each** of the four `docs/` subfolders, ask the user:

"Which docs/ subfolders should be added to .gitignore?"

Present as multi-select:
- `docs/sessions/` — session logs (often contain verbose context)
- `docs/decisions/` — ADRs (usually worth committing)
- `docs/research/` — research spikes (usually worth committing)
- `docs/plans/` — implementation plans (usually worth committing)

Append chosen entries to `.gitignore`. If `.gitignore` doesn't exist, create it.

### Step 6: Legacy Migration (if applicable)

If `.claude/memory/session-logs/` exists and has files:

Ask: "Found existing session logs in `.claude/memory/session-logs/`. Move them to `docs/sessions/`?"

If yes:
```bash
mv .claude/memory/session-logs/*.md docs/sessions/
```

### Step 7: Confirm

Print summary:

```
Workspace initialized:

  Created:
  - docs/sessions/
  - docs/decisions/
  - docs/research/
  - docs/plans/
  - WORK.md

  .gitignore:
  - docs/sessions/ (ignored)

  Migration:
  - Moved 3 files from .claude/memory/session-logs/ to docs/sessions/

Ready to use /start and /end for session management.
```

## Important

- This skill is **idempotent** — running `/init` twice should not break anything
- Use `{cwd}` (from `pwd`) for all paths, never hardcode
- If `.gitignore` already has an entry, don't duplicate it
- The skill creates structure only — it does not populate content beyond the WORK.md template

## Tools Usage

- **Bash**: `mkdir -p`, `ls`, `mv`, `pwd`, `date`
- **Write**: Create WORK.md, append to .gitignore
- **Read**: Check existing .gitignore content
- **Glob**: Detect existing structure
- **AskUserQuestion**: Smart merge decisions, .gitignore choices
