---
name: start
description: Begin a work session — reads current state, shows active tracks, validates handoff quality, presents onboarding summary. Auto-invokes on "/start", "start session".
allowed-tools: [Read, Glob, Bash, AskUserQuestion]
---

# Start — Session Onboarding

Reads current project state and onboards you into a new session. This is a **read-only** skill — it does not modify any files.

## When to Auto-Invoke

Trigger when user says:
- `/start`
- "start session"
- "begin session"
- "pick up where we left off"
- "what were we working on?"

## How It Works

### Step 1: Read State Files

Read these files from the project root:
1. `WORK.md` — structured AI context (multi-track state)
2. `TODO.md` — human scratchpad (if it exists)

If `WORK.md` does not exist:
- Say: "No WORK.md found. Run `/init` to set up the workspace first."
- Stop here.

### Step 2: Find Latest Session Log(s)

```bash
ls -t docs/sessions/ | head -5
```

Read the most recent session log. If `docs/sessions/` is empty or doesn't exist, note this as the first session.

### Step 3: Present Track Overview

Show all tracks from WORK.md with their status:

```
**Your tracks:**
  🔴 Frontend UI — last worked 2026-03-21 (active)
  🟡 Eval Framework — last worked 2026-03-20 (active)
  ⚪ Marketing — paused, waiting on DSP quality

**Open questions:**
  - [from WORK.md Open Questions section]

**Active plan:** [from WORK.md Active Plan field, or "none"]
```

Use emoji to indicate status:
- 🔴 Active (recently worked on)
- 🟡 Active (not recently worked on)
- ⚪ Paused or blocked
- 🔵 New (just created)

### Step 4: Show TODO.md Highlights

If TODO.md exists, show the most recent date entry (just the last block, not the whole file):

```
**Your latest notes (from TODO.md):**
  ## 2026-03-21
  - audition 12 ref WAVs — do this before optimizer
  - optimizer design: grid vs random vs bayesian?
```

This gives the user their own context back quickly.

### Step 5: Show Last Session Details

For the most recent session log, validate the "Next Session Starts With" field against all four criteria:

1. **Starts with a verb** — "Run...", "Build...", "Test...", "Wire up..."
2. **Names a specific thing** — file path, tool name, command, or concrete question
3. **Is self-contained** — a new session reading only this line knows what to do
4. **Captures half-done context** — if something was mid-task, says where it was left

If the handoff is clean:
```
**Last session ([date]):** [track name]
  [2-4 bullets from Done section]

**Resuming from:**
  "[paste Next Session Starts With from session log]"
```

If the handoff is incomplete (missing or vague):
```
**Incomplete handoff detected**
  Last session log is missing a clear "Next Session Starts With" field.
  Falling back to WORK.md track states.
```

If this is the first session:
```
**First session detected**
  No prior session logs found. Starting fresh from WORK.md.
```

### Step 6: Ask What to Focus On

Ask: "Which track(s) are you working on this session?"

Present options based on tracks + last session handoff:

```
1. Resume: "[Next Session Starts With summary]" (🔴 Frontend UI)
2. 🟡 Eval Framework — audition refs, then build optimizer
3. ⚪ Marketing — currently paused
4. Start a new track
```

**Record the user's track selection** — this determines which track(s) `/end` will update later. Mention this: "Got it — I'll scope this session to [track name]. `/end` will only update that track in WORK.md."

## Important

- This is a **read-only** skill — it never writes or modifies files
- Show the track overview first, then details — let the user orient quickly
- Show TODO.md highlights to restore the user's own mental context
- All paths are relative to the project root
- If WORK.md and session log disagree on state, prefer the session log (more recent)
- **Declare the active track(s)** — this scopes what `/end` will update

## Tools Usage

- **Read**: Read WORK.md, TODO.md, and session log files
- **Glob**: Find session log files in docs/sessions/
- **Bash**: List and sort session logs by date (`ls -t`)
- **AskUserQuestion**: Ask which track(s) to work on
