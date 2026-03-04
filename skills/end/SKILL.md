---
name: end
description: End a work session — scans conversation, writes session log, updates WORK.md, quality-tests handoff. Auto-invokes on "/end", "end session", "close session", "wrap up".
allowed-tools: [Read, Write, Grep, Glob, Bash, AskUserQuestion]
---

# End — Session Close & Handover

Closes the current session with a quality-tested handoff to the next session.

## When to Auto-Invoke

Trigger when user says:
- `/end`
- "end session" / "close session"
- "wrap up" / "let's wrap up"
- "save and close"
- "done for today"

## How It Works

### Step 1: Scan the Conversation

Review the full conversation. Also discover artifacts:

```bash
# What commits were made this session?
git log --oneline --since="6 hours ago" --no-merges 2>/dev/null
# What files changed?
git diff HEAD~10..HEAD --stat 2>/dev/null
```

Identify:
- What was **completed** this session
- What was **started but not finished** (half-done work, where it was left)
- Any **decisions made** that aren't yet in an ADR
- Any **research conducted** that should be saved
- Any **new open questions** that came up
- What the logical **next concrete action** is

### Step 2: Confirm with the User

Present a summary and wait for confirmation:

```
Here's what I captured from this session:

**Done:**
- item
- item

**Half-done / in-progress:**
- item (state: where it was left)

**Deferred:**
- item — why

**Decisions made:**
- decision

**New open questions:**
- question

**Next Session Starts With:**
"[proposed handoff note]"

Does this look right? Anything to add or change?
```

Wait for user confirmation before writing any files.

### Step 3: Write the Session Log

Create `docs/sessions/YYYY-MM-DD-topic.md` using today's date and a topic derived from the work done.

Template:

```markdown
# Session: YYYY-MM-DD — [Topic]

## Goal
[What this session set out to accomplish]

## Done
- [x] [Completed item]

## Half-Done
- [ ] [In-progress item] — [current state, what remains]

## Deferred
- [ ] [Punted item] — [reason]

## Decisions Made
- **[Decision]**: [rationale]
- ADR: [filename in docs/decisions/ if written, or "none"]

## Next Session Starts With
[THE MOST IMPORTANT FIELD — must pass quality test below]
```

If multiple sessions happen on the same date, append a suffix: `YYYY-MM-DD-topic-2.md`.

### Step 4: Quality-Test "Next Session Starts With"

Before writing, verify the field passes **ALL** of these criteria:

1. **Starts with a verb** — "Run...", "Build...", "Test...", "Ask user..."
   - NOT "Continue working on..."
2. **Names a specific thing** — file path, tool name, command, or question
   - NOT a vague area like "the feature"
3. **Is self-contained** — a new Claude session reading only this line knows what to do without asking "what does that mean?"
4. **Captures any half-done context** — if something was mid-task, says where it was left and what the next step is

**Bad**: "Continue building the auth feature"
**Good**: "Open `src/auth/webhook.ts` — the Stripe webhook route exists but `constructEvent` is stubbed out. Next: implement signature verification using `STRIPE_WEBHOOK_SECRET` from env, then add handlers for `checkout.session.completed` and `invoice.payment_failed` events"

If the draft doesn't pass, rewrite it until it does. If unsure, ask the user for help refining it.

### Step 5: Update WORK.md

Read the current `WORK.md` and update these fields:

- **Last updated**: today's date
- **Current Phase**: ask user if phase changed, or infer from work done
- **Just Done**: from this session's "Done" list
- **Blocked / Open Questions**: from session context (new questions + any unresolved items)
- **Next Steps**: derived from "Next Session Starts With" + deferred items (first item matches the handoff note)
- **Active Files**: from files touched this session (use git diff if available)
- **Active Plan**: preserve existing value, or update if a plan was created/completed

If WORK.md doesn't exist, create it from the template (see `/init` skill).

### Step 6: Optional Follow-ups

Check if any follow-up artifacts should be written:

- **Decision made without ADR?** → Ask: "Want me to write an ADR to `docs/decisions/YYYY-MM-DD-topic.md`?"
- **Research conducted without notes?** → Ask: "Want me to save research notes to `docs/research/YYYY-MM-DD-topic.md`?"
- **Half-done work?** → Confirm it's captured clearly in both session log and WORK.md

These are opt-in via AskUserQuestion, not automatic.

### Step 7: Final Output

Print this clearly so the user sees it at a glance:

```
Session closed.

  Session log: docs/sessions/YYYY-MM-DD-topic.md
  WORK.md: Updated

Next session starts with:
"[paste the exact Next Session Starts With line here]"
```

This is the last thing the user sees — make it prominent.

## Session Log Guidelines

- **One template** — keep it simple. The session type (implementation, research, planning) doesn't change the template. Rich artifacts go to their own `docs/` subfolders.
- **Immutable** — once written, session logs are never edited. If corrections are needed, note them in the next session.
- **Filename**: `YYYY-MM-DD-kebab-case-topic.md` (lowercase, dashes, 3-5 words max)

## Tools Usage

- **Bash**: `git log`, `git diff`, `pwd`, `date`, `ls -t docs/sessions/`
- **Read**: Read existing WORK.md, check for prior session logs
- **Write**: Write session log, update WORK.md
- **Grep**: Search for related sessions or patterns
- **Glob**: Find files in docs/ subfolders
- **AskUserQuestion**: Confirm summary, offer ADR/research follow-ups, refine handoff note

## Key Principles

1. **Handoff quality is non-negotiable** — the "Next Session Starts With" field must pass all four criteria
2. **Confirm before writing** — always show the summary and wait for user approval
3. **Comprehensive but concise** — capture what matters, skip the noise
4. **Artifact-grounded** — reference git commits, file paths, concrete evidence
5. **User controls** — they approve the summary, decide on follow-up ADRs/research notes
