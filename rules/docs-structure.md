# Documentation Structure

## Overview

Projects using this framework organize documentation under `docs/` at the project root, with `WORK.md` as the live state file.

## Folder Layout

| Folder | Purpose | Written By |
|--------|---------|-----------|
| `docs/sessions/` | Session logs — execution history of each work session | `/end` skill |
| `docs/decisions/` | Architecture Decision Records (ADRs) — any decision worth remembering | Any skill, anytime |
| `docs/research/` | Research spikes, technical investigations, feasibility studies | `/research` |
| `docs/plans/` | Implementation plans (lightweight and comprehensive) | `/plan` |

## WORK.md

`WORK.md` lives at the project root. It is the **live state file** — the single place to look for what is happening right now.

Updated by `/end` at session close. Read by `/start` at session open.

Sections:
- **Current Phase** — where the project is
- **Just Done** — last session's accomplishments
- **Blocked / Open Questions** — anything unresolved
- **Next Steps** — prioritized next actions (first matches "Next Session Starts With")
- **Active Files** — files currently in-flight
- **Active Plan** — path to current plan in `docs/plans/`

## Where to Write What

- **Session closed?** → `docs/sessions/`
- **Decision made?** → `docs/decisions/`
- **Research spike done?** → `docs/research/`
- **Plan created or updated?** → `docs/plans/`
- **Live project state?** → `WORK.md`

## Decisions Deserve Their Own Record

`docs/decisions/` is the most underused folder. **Any time a meaningful decision is made — write an ADR.** This applies regardless of which skill or context the decision came from:

- During `/research` — if the research leads to choosing Option A over B, write an ADR to `docs/decisions/`
- During `/c-suite` — the meeting output goes to `docs/decisions/`
- During `/end` — if the session involved decisions, offer to write ADRs
- During `/plan` — if planning requires choosing an approach, write an ADR
- During normal work — if you and the user decide on something significant (tech stack, architecture pattern, library choice, trade-off acceptance), write an ADR

**Why this matters**: Decisions get lost in session logs and conversation history. ADRs are the only artifact specifically designed to capture *why* something was chosen and *what was rejected*. Without them, future sessions re-litigate the same decisions.

**ADR template** (simple):
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
