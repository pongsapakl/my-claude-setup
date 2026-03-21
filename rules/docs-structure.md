# Documentation Structure

## Overview

Projects using this framework organize documentation under `docs/` at the project root, with three key files:

| File | Purpose | Audience |
|------|---------|----------|
| `TODO.md` | Human scratchpad — freeform task tracking, dated entries, ideas | **Human** |
| `WORK.md` | AI context — structured multi-track state for session continuity | **AI** |
| `docs/sessions/*.md` | Rich narrative session logs — immutable archive | **Both** |

## Folder Layout

| Folder | Purpose | Written By |
|--------|---------|-----------|
| `docs/sessions/` | Session logs — rich narrative execution history of each work session | `/end` skill |
| `docs/decisions/` | Architecture Decision Records (ADRs) — any decision worth remembering | **Any skill, anytime** |
| `docs/research/` | Research spikes, technical investigations, feasibility studies | `/research`, any skill |
| `docs/plans/` | Implementation plans (lightweight and comprehensive) | `/plan` |

## WORK.md — AI Context File

`WORK.md` lives at the project root. It is the **AI-facing** structured state file for session continuity.

Updated by `/end` at session close (track-scoped merge). Read by `/start` at session open.

### Multi-Track Structure

WORK.md uses a track-based layout so parallel workstreams don't clobber each other:

```markdown
## Tracks

### Track Name
**Status**: Active | Paused | Blocked
**Last session**: date — link to session log
**State**: what's done, what's in progress
**Next**: specific next action
**Key files**: files being worked on
```

**Critical rule**: `/end` only updates the track(s) that were active. Other tracks are left untouched.

### Sections

- **Tracks** — independent workstream blocks (each with Status, State, Next, Key Files)
- **Open Questions** — append-only; new questions added, only explicitly-answered ones removed
- **Active Plan** — path to current plan in `docs/plans/`

## TODO.md — Human Scratchpad

`TODO.md` lives at the project root. It is the **human-facing** freeform tracking file.

- Dated entries as headers (`## YYYY-MM-DD`)
- Strikethrough for done (`~~completed item~~`) — never delete old entries
- Raw thoughts, brainstorming, and ideas welcome
- An "Active Tracks" section at the top for quick orientation
- `/end` appends a new dated block but never modifies existing content
- The user may also edit this file directly at any time

**Design philosophy**: This file follows the style of a personal TODO list — minimal structure, maximum scannability. It's what the user glances at to remember where they are.

## Where to Write What

- **Session closed?** → `docs/sessions/`
- **Decision made?** → `docs/decisions/` **(always — see below)**
- **Research spike done?** → `docs/research/` — and if it led to a decision, **also** write an ADR to `docs/decisions/`
- **Plan created or updated?** → `docs/plans/` — and if planning required choosing an approach, **also** write an ADR
- **Live AI state?** → `WORK.md` (track-scoped)
- **Human task tracking?** → `TODO.md` (append-only)

## Decisions Deserve Their Own Record

`docs/decisions/` is the most important folder for long-term project health, yet the easiest to forget. **Any time a meaningful decision is made — write an ADR.** This is not limited to specific skills. It applies everywhere:

- During `/research` — if the research concludes with "use Option A over B", the research note goes to `docs/research/` AND an ADR goes to `docs/decisions/`
- During `/c-suite` — the meeting output goes to `docs/decisions/`
- During `/end` — if the session involved decisions, **default to drafting ADRs** (user opts out, not in)
- During `/plan` — if planning requires choosing an approach, write an ADR
- **During normal work** — if you and the user decide on something significant (tech stack, architecture, library choice, trade-off acceptance), proactively offer to write an ADR. Don't wait for a skill to prompt you.

**Why this matters**: Decisions get lost in session logs and conversation history. ADRs are the only artifact specifically designed to capture *why* something was chosen and *what was rejected*. Without them, future sessions re-litigate the same decisions.

**When to write an ADR**:
- Chose one library/framework/tool over another
- Decided on an architecture pattern or data model
- Accepted a trade-off ("use X even though Y is faster because...")
- Changed direction from a previous decision (write a new ADR that supersedes the old one)
- Any "should we X or Y?" discussion that reached a conclusion

**When NOT to write an ADR**:
- Trivial choices (variable naming, minor formatting)
- Decisions already fully documented in a plan

**ADR template** (keep it simple — a short ADR is better than no ADR):
```markdown
# [Short Title]
Date: YYYY-MM-DD
Status: Accepted

## Context
[What prompted this decision]

## Decision
[What we decided]

## Reasons
[Why this over alternatives]

## Rejected Alternatives
[What was considered and why it was not chosen]
```

## Mid-Session Documentation

Don't wait for session end to capture artifacts. When a decision or research spike happens mid-session, prompt immediately:

- Context is fresher (pre-compact)
- User is still in "working" mode, not "wrapping up" mode
- The detail captured will be richer

**Default-write, not default-skip**: Draft the ADR or research doc and show it to the user. It's easier to say "skip" than to reconstruct lost context.

## File Naming Convention

All docs use: `YYYY-MM-DD-kebab-case-topic.md`

Examples:
- `docs/sessions/2026-03-04-auth-webhook-setup.md`
- `docs/decisions/2026-03-04-stripe-over-paypal.md`
- `docs/research/2026-03-04-realtime-audio-options.md`
- `docs/plans/2026-03-04-user-authentication.md`

## Important

- Use relative paths from project root (e.g., `docs/plans/2026-03-04-auth-flow.md`)
- Do **not** write session/decision/research/plan docs to `.claude/memory/` — use `docs/` subfolders
- Run `/init` to bootstrap the folder structure for a new project
