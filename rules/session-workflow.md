# Session Workflow

## Lifecycle

Every working session follows this flow:

1. **`/start`** — Read WORK.md + TODO.md + latest session log, show track overview, ask which track to focus on
2. **Work** — Normal development, research, planning, meetings
3. **`/end`** — Scan conversation, write rich session log, merge-update active track in WORK.md, append to TODO.md

## Three Files, Three Audiences

| File | Audience | Style | Updated By |
|------|----------|-------|------------|
| `TODO.md` | **Human** | Freeform scratchpad, dated entries, strikethrough for done | User writes, `/end` appends (never deletes) |
| `WORK.md` | **AI** | Structured multi-track state | `/end` merge-updates per track (never full-replaces) |
| `docs/sessions/*.md` | **Both** | Rich narrative archive | `/end` creates (immutable once written) |

## Multi-Track Work

Projects often have parallel workstreams (frontend, backend, infra, marketing, etc.). The system supports this:

- `WORK.md` has a `## Tracks` section with independent track blocks
- `/start` shows all tracks and asks which to focus on
- `/end` only updates the track(s) that were active — other tracks stay untouched
- Each track has its own Status, State, Next, and Key Files

This prevents the "last session wins" problem where one workstream overwrites another's state.

## Track-Scoped Updates (Critical Rule)

When `/end` updates WORK.md:
- **Only modify the active track(s)** declared at `/start`
- **Append to Open Questions** — add new, only remove those explicitly answered
- **Never delete other tracks' state**
- **Never replace the entire file**

## TODO.md as Human Scratchpad

TODO.md is the user's personal tracking file, designed for quick human scanning:
- Dated entries as headers (`## YYYY-MM-DD`)
- Strikethrough (`~~done item~~`) for completed work — never delete old entries
- Raw thoughts, brainstorming, and ideas welcome
- `/end` appends a new dated block but never modifies existing content
- The user may also edit this file directly at any time

## "Next Session Starts With" — Quality Criteria

The "Next Session Starts With" field in session logs is the bridge between sessions. It must pass **all four criteria**:

1. **Starts with a verb** — "Implement...", "Fix...", "Test...", "Wire up..."
   - NOT: "Continue working on..."
2. **Names a specific thing** — file path, command, tool, concrete question
   - NOT: "the auth module" or "some stuff"
3. **Self-contained** — a new session can act on it without reading the full log
4. **Half-done aware** — if work is mid-stream, says exactly where it was left and what comes next

## Session Log Richness

Session logs should be **detailed and narrative**, not just checkboxes. They serve as the institutional memory of the project. A future session (or human) should be able to read a session log and understand:

- What was tried and why
- What worked and what didn't
- What decisions were made and what alternatives were considered
- What was surprising or took longer than expected
- Technical details that would help someone pick up the work

**When in doubt, include more detail rather than less.** A 200-line session log with rich narrative is more valuable than a 30-line checklist.

## Mid-Session Documentation

Don't wait for `/end` to capture decisions and research. When something significant happens during a session:

- **Decision made?** → Proactively offer to write an ADR immediately while context is fresh
- **Research spike done?** → Offer to save findings to `docs/research/` now
- **Plan created?** → Save to `docs/plans/` now

Context degrades over time (especially after `/compact`). Capturing artifacts mid-session preserves richer detail than reconstructing them at `/end`.

## When to Use Each Skill

| Situation | Skill |
|-----------|-------|
| Starting a session | `/start` |
| Ending a session | `/end` |
| Creating an implementation plan | `/plan` |
| Investigating a topic | `/research` |
| Making a strategic decision | `/c-suite` |
| Setting up a new project | `/init` |
