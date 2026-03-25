---
name: init
description: Bootstrap workspace with docs structure, WORK.md, and TODO.md. Auto-invokes on "/init", "initialize workspace", "set up project structure".
allowed-tools: [Bash, Write, Read, Glob, AskUserQuestion]
---

# Init — Workspace Bootstrapper

Sets up the documentation structure, live state file (WORK.md), and human scratchpad (TODO.md) for a new project.

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
# Check for WORK.md and TODO.md
ls WORK.md TODO.md 2>/dev/null
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

- "TODO.md already exists. What should I do?"
  - **Keep** (leave as-is)
  - **Replace** (overwrite with fresh template)
  - **Skip** (don't touch)

If nothing exists, skip this step entirely — just create everything.

### Step 3: Create Folder Structure

For each folder the user approved (or that doesn't exist yet):

```bash
mkdir -p docs/sessions docs/decisions docs/research docs/plans
```

### Step 4: Create WORK.md (AI Context File)

Only if it doesn't exist or user chose "Replace".

WORK.md is the **AI-facing** structured state file. It uses a multi-track format so parallel workstreams don't clobber each other.

Use this template:

```markdown
# WORK.md

> Last updated: {YYYY-MM-DD}

## Tracks

### {Track Name}
**Status**: Active | Paused | Blocked
**Last session**: {date} — {link to session log}
**State**: {Current state of this track — what's done, what's in progress}
**Next**: {Specific next action for this track}
**Key files**: {Files actively being worked on in this track}

## Open Questions
- {Any blockers or questions that span tracks}

## Active Plan
None
```

Replace `{YYYY-MM-DD}` with today's date. Leave placeholder text for user to fill in.

Ask the user: "What are your initial work tracks? (e.g., 'frontend', 'backend', 'infra') — or just one track is fine too."

### Step 5: Create TODO.md (Human Scratchpad)

Only if it doesn't exist or user chose "Replace".

TODO.md is the **human-facing** freeform scratchpad. It is append-only — old entries are never deleted.

Use this template:

```markdown
# TODO

## Active Tracks
- {emoji} {Track name} ({brief description})

## {YYYY-MM-DD}
- {What needs to happen today/next}
```

Replace `{YYYY-MM-DD}` with today's date. Keep it minimal — the user will fill this in their own style.

### Step 6: Configure .gitignore

For **each** of the four `docs/` subfolders, ask the user:

"Which docs/ subfolders should be added to .gitignore?"

Present as multi-select:
- `docs/sessions/` — session logs (often contain verbose context)
- `docs/decisions/` — ADRs (usually worth committing)
- `docs/research/` — research spikes (usually worth committing)
- `docs/plans/` — implementation plans (usually worth committing)

Append chosen entries to `.gitignore`. If `.gitignore` doesn't exist, create it.

### Step 7: Install Plugin Rules (Optional)

Ask the user:

"Install plugin rules to `.claude/rules/`? These provide session workflow, docs structure, git commit, security, and other conventions. (Yes / No)"

**If Yes:**

1. Create `.claude/rules/` if it doesn't exist:
```bash
mkdir -p .claude/rules
```

2. Copy each rule file from the plugin's `rules/` directory. For each file, prepend a deprecation header comment, then write to the target project:

Files to copy:
- `cli-workflow.md`
- `discussion-protocol.md`
- `docs-structure.md`
- `git-commit-workflow.md`
- `security-standards.md`
- `session-workflow.md`

Each copied file must start with this comment block:

```markdown
<!-- Installed by my-claude-team plugin.
     Native plugin rules support is expected in Claude Code — once available,
     these files can be removed and rules will load automatically from the plugin. -->

```

Then append the original rule content below it.

Use `Read` to read each rule from the plugin directory, then `Write` to create the file in `.claude/rules/`.

**If No:** Skip this step entirely.

### Step 8: Legacy Migration (if applicable)

If `.claude/memory/session-logs/` exists and has files:

Ask: "Found existing session logs in `.claude/memory/session-logs/`. Move them to `docs/sessions/`?"

If yes:
```bash
mv .claude/memory/session-logs/*.md docs/sessions/
```

### Step 9: Confirm

Print summary:

```
Workspace initialized:

  Created:
  - docs/sessions/
  - docs/decisions/
  - docs/research/
  - docs/plans/
  - WORK.md (AI context — multi-track state)
  - TODO.md (your scratchpad — freeform, append-only)

  .gitignore:
  - docs/sessions/ (ignored)

  Rules (if installed):
  - .claude/rules/cli-workflow.md
  - .claude/rules/discussion-protocol.md
  - .claude/rules/docs-structure.md
  - .claude/rules/git-commit-workflow.md
  - .claude/rules/security-standards.md
  - .claude/rules/session-workflow.md
  (Note: these can be removed once native plugin rules support lands)

  Migration:
  - Moved 3 files from .claude/memory/session-logs/ to docs/sessions/

Ready to use /start and /end for session management.

File guide:
  TODO.md → Your notes. You own it. Claude appends, never deletes.
  WORK.md → AI context. /end updates per-track, never overwrites other tracks.
  docs/sessions/ → Rich session logs (immutable archive).
```

## Important

- This skill is **idempotent** — running `/init` twice should not break anything
- Use `{cwd}` (from `pwd`) for all paths, never hardcode
- If `.gitignore` already has an entry, don't duplicate it
- The skill creates structure only — it does not populate content beyond templates

## Tools Usage

- **Bash**: `mkdir -p`, `ls`, `mv`, `pwd`, `date`
- **Write**: Create WORK.md, TODO.md, append to .gitignore
- **Read**: Check existing .gitignore content
- **Glob**: Detect existing structure
- **AskUserQuestion**: Smart merge decisions, .gitignore choices, track names
