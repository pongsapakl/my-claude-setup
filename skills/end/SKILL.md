---
name: end
description: End a work session — scans conversation, writes rich session log, merges track state into WORK.md (without overwriting other tracks), appends to TODO.md. Auto-invokes on "/end", "end session", "wrap up".
allowed-tools: [Read, Write, Grep, Glob, Bash, AskUserQuestion]
---

# End — Session Close & Handover

Closes the current session with a rich, narrative session log and track-scoped updates to WORK.md. Never overwrites information from other sessions or tracks.

## When to Auto-Invoke

Trigger when user says:
- `/end`
- "end session" / "close session"
- "wrap up" / "let's wrap up"
- "save and close"
- "done for today"

## How It Works

### Step 1: Identify Active Track(s)

Check what track(s) were declared at `/start`. If `/start` wasn't used, ask:

"Which track(s) did you work on this session?"

This is critical — it determines which parts of WORK.md get updated. **Other tracks are left untouched.**

### Step 2: Scan the Conversation

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
- Any **decisions made** — with alternatives considered and rationale
- Any **research conducted** that should be saved
- Any **new open questions** that came up
- **Technical details** worth preserving (commands, error messages, approaches tried)
- **Key learnings or surprises** — things that took longer than expected, unexpected findings
- What the logical **next concrete action** is

### Step 3: Confirm with the User

Present a summary and wait for confirmation:

```
Here's what I captured from this session (Track: {track name}):

**What happened:**
[2-3 sentence narrative of the session — the "story"]

**Done:**
- item
- item

**Half-done / in-progress:**
- item (state: where it was left)

**Deferred:**
- item — why

**Decisions made:**
- decision — chose X over Y because Z

**Key learnings:**
- surprising finding or insight

**Next Session Starts With:**
"[proposed handoff note]"

Does this look right? Anything to add or change?
```

Wait for user confirmation before writing any files.

### Step 4: Write the Session Log (Rich Format)

Create `docs/sessions/YYYY-MM-DD-topic.md` using today's date and a topic derived from the work done.

Template:

```markdown
# Session: YYYY-MM-DD — [Topic]

**Track(s):** [which track(s) this session worked on]

## Goal
[What this session set out to accomplish — 1-3 sentences]

## What Happened
[Freeform narrative of the session. What was tried, what worked, what didn't,
key turning points. This is the "story" of the session — write as much detail
as needed. Future sessions depend on this context.

Include:
- Approaches tried and why they did or didn't work
- Error messages encountered and how they were resolved
- Key realizations that changed direction
- Commands or techniques that proved useful

This section should be the richest part of the log. A future session (or human)
reading this should understand not just WHAT was done but HOW and WHY.]

## Done
- [x] [Completed item — with brief context]
- [x] [Completed item — reference commit if applicable]

## Half-Done
- [ ] [In-progress item] — [current state: what's done, what remains, where to pick up]

## Deferred
- [ ] [Punted item] — [reason for deferral]

## Decisions Made
For each significant decision:
- **Decision**: [what was decided]
  - **Why**: [rationale]
  - **Alternatives considered**: [what else was evaluated, why rejected]
  - **Trade-offs accepted**: [what we're giving up]
  - ADR: [filename in docs/decisions/ if written, or "none — not significant enough"]

## Technical Details
[Code changes worth noting, architecture patterns established, commands that worked,
configurations that matter. Include code snippets where they help a future session.
Reference specific files and line numbers.]

## Key Learnings / Surprises
- [Anything unexpected discovered]
- [Things that took longer or shorter than expected, and why]
- [Patterns or techniques worth remembering]

## Next Session Starts With
[THE MOST IMPORTANT FIELD — must pass quality test below]
```

If multiple sessions happen on the same date, append a suffix: `YYYY-MM-DD-topic-2.md`.

### Step 5: Quality-Test "Next Session Starts With"

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

### Step 6: Update WORK.md (Track-Scoped Merge)

**CRITICAL: Only update the track(s) that were active this session. Never touch other tracks.**

Read the current `WORK.md`. Then:

1. **Update only the active track's block:**
   - **Status**: update if changed
   - **Last session**: today's date + link to session log
   - **State**: from this session's done/half-done items
   - **Next**: from "Next Session Starts With"
   - **Key files**: from files touched this session

2. **Merge into Open Questions** (append new, don't delete existing):
   - Add new questions from this session
   - Only remove questions that were explicitly answered this session

3. **Update Active Plan**: preserve existing value, or update if a plan was created/completed

4. **Update the "Last updated" date**

**DO NOT:**
- Replace the entire file
- Delete or modify other tracks
- Remove open questions from other sessions
- Change Active Plan unless this session specifically addressed it

If a track doesn't exist yet in WORK.md (new workstream), add it as a new `### Track` block.

### Step 7: Append to TODO.md

If `TODO.md` exists, **append** a new dated block. Never delete or modify existing entries.

Read TODO.md first. Then append after the most recent date entry (or after "## Active Tracks" if no date entries yet):

```markdown
## {YYYY-MM-DD}
- ~~{completed item}~~
- ~~{completed item}~~
- {next action from handoff}
- {any new open question or idea}
```

If the "Active Tracks" section exists, update track statuses (add new tracks, update emoji status). But never delete tracks from this list.

If `TODO.md` doesn't exist, skip this step (don't create it — that's `/init`'s job).

### Step 8: Trigger Documentation Generation

Check if any follow-up artifacts should be written. **Default to writing them** — the user opts out rather than opting in:

- **Decision made without ADR?** → "I'll draft an ADR to `docs/decisions/YYYY-MM-DD-topic.md` — here's what I'll write: [preview]. Should I save it?"
- **Research conducted without notes?** → "I'll save research notes to `docs/research/YYYY-MM-DD-topic.md` — here's the summary: [preview]. Should I save it?"

**Why default-write**: By session end, context is at its richest. If the user skips now, the detail is lost forever. It's easier to delete an unwanted doc than to reconstruct a lost one.

### Step 9: Final Output

Print this clearly so the user sees it at a glance:

```
Session closed.

  Track: {track name}
  Session log: docs/sessions/YYYY-MM-DD-topic.md
  WORK.md: Updated ({track name} track only)
  TODO.md: Appended
  ADR: docs/decisions/YYYY-MM-DD-topic.md (if written)

Next session starts with:
  "{paste the exact Next Session Starts With line here}"
```

This is the last thing the user sees — make it prominent.

## Session Log Guidelines

- **Rich and narrative** — write for a future session (or human) that has zero context. Include the "why" and "how", not just the "what". When in doubt, include more detail rather than less.
- **Immutable** — once written, session logs are never edited. If corrections are needed, note them in the next session.
- **Filename**: `YYYY-MM-DD-kebab-case-topic.md` (lowercase, dashes, 3-5 words max)

## Tools Usage

- **Bash**: `git log`, `git diff`, `pwd`, `date`, `ls -t docs/sessions/`
- **Read**: Read existing WORK.md, TODO.md, check for prior session logs
- **Write**: Write session log, merge-update WORK.md, append to TODO.md
- **Grep**: Search for related sessions or patterns
- **Glob**: Find files in docs/ subfolders
- **AskUserQuestion**: Confirm summary, track selection, ADR/research follow-ups

## Key Principles

1. **Handoff quality is non-negotiable** — the "Next Session Starts With" field must pass all four criteria
2. **Confirm before writing** — always show the summary and wait for user approval
3. **Rich and narrative** — detailed logs are a feature, not a bug. Future sessions depend on this context. Capture the story, not just the checkboxes.
4. **Track-scoped updates** — only update the tracks you worked on. Never overwrite other tracks.
5. **Append-only TODO.md** — never delete the user's notes. Strikethrough for done, append for new.
6. **Default-write artifacts** — ADRs and research docs are drafted by default. User opts out, not in.
7. **Artifact-grounded** — reference git commits, file paths, concrete evidence
8. **User controls** — they approve the summary, decide on follow-up artifacts
