---
name: start
description: Begin a work session — reads current state, validates handoff quality, presents onboarding summary. Auto-invokes on "/start", "start session".
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

### Step 1: Read WORK.md

Read `WORK.md` from the project root.

If `WORK.md` does not exist:
- Say: "No WORK.md found. Run `/init` to set up the workspace first."
- Stop here.

### Step 2: Find Latest Session Log

```bash
ls -t docs/sessions/ | head -1
```

Read the most recent file. If `docs/sessions/` is empty or doesn't exist, note this as the first session.

### Step 3: Validate Handoff Quality

Check the session log's "Next Session Starts With" field against all four criteria:

1. **Starts with a verb** — "Run...", "Build...", "Test...", "Wire up..."
2. **Names a specific thing** — file path, tool name, command, or concrete question
3. **Is self-contained** — a new session reading only this line knows what to do
4. **Captures half-done context** — if something was mid-task, says where it was left

If any criterion fails, flag it — do not silently use a vague handoff.

### Step 4: Present Onboarding Summary

**If handoff is clean**, use this format:

```
**Phase:** [from WORK.md Current Phase]

**Last session ([date from filename]):**
- [2-4 bullets from session log Done section]

**Resuming from:**
"[paste Next Session Starts With from session log]"

**Open questions / blockers:**
- [from WORK.md Blocked / Open Questions, or "none"]

**Active plan:**
[from WORK.md Active Plan field, or "none"]

**Other next steps:**
1. [from WORK.md Next Steps, skip the first if already covered by Resuming from]
2. ...
```

**If handoff is incomplete** (missing or vague "Next Session Starts With"):

```
**Incomplete handoff detected**

The last session log is missing a clear "Next Session Starts With" field.
Falling back to WORK.md Next Steps — confirm these are still accurate before starting.

**Phase:** [from WORK.md]
**Next Steps:** [from WORK.md]
**Open questions:** [from WORK.md]
**Active plan:** [from WORK.md]
```

**If this is the first session** (no session logs):

```
**First session detected**

No prior session logs found. Here is the current state from WORK.md:

**Phase:** [from WORK.md]
**Next Steps:** [from WORK.md]
**Active plan:** [from WORK.md]
```

### Step 5: Ask What to Focus On

Ask: "What would you like to focus on this session?"

If the handoff was clean, offer options:
1. Resume from: "[Next Session Starts With summary]"
2. [Other next steps from WORK.md]

If the handoff was incomplete, list WORK.md Next Steps and ask user to confirm which is still relevant.

## Important

- This is a **read-only** skill — it never writes or modifies files
- Keep the onboarding summary short — goal is fast context restoration in under 30 seconds of reading
- All paths are relative to the project root
- If both WORK.md and session log disagree on state, prefer the session log (it's more recent and detailed)

## Tools Usage

- **Read**: Read WORK.md and session log files
- **Glob**: Find session log files in docs/sessions/
- **Bash**: List and sort session logs by date (`ls -t`)
- **AskUserQuestion**: Ask what to focus on
