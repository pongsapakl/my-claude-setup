# Session Workflow

## Lifecycle

Every working session follows this flow:

1. **`/start`** — Read WORK.md + latest session log, validate handoff, present onboarding summary, ask what to focus on
2. **Work** — Normal development, research, planning, meetings
3. **`/end`** — Scan conversation, write session log, update WORK.md, quality-test handoff

## WORK.md as Live State

`WORK.md` is the project's single source of truth for current status.

- Updated at the end of every session by `/end`
- Read at the start of every session by `/start`
- References the active plan in `docs/plans/`

## "Next Session Starts With" — Quality Criteria

The "Next Session Starts With" field in session logs is the bridge between sessions. It must pass **all four criteria**:

1. **Starts with a verb** — "Implement...", "Fix...", "Test...", "Wire up..."
   - NOT: "Continue working on..."
2. **Names a specific thing** — file path, command, tool, concrete question
   - NOT: "the auth module" or "some stuff"
3. **Self-contained** — a new session can act on it without reading the full log
4. **Half-done aware** — if work is mid-stream, says exactly where it was left and what comes next

**Bad**: "Continue working on the project"
**Good**: "Wire up the Stripe webhook handler in `api/webhooks/stripe.ts` — the route exists but signature verification is stubbed out. Next: implement the `constructEvent` call using the webhook secret from env"

## When to Use Each Skill

| Situation | Skill |
|-----------|-------|
| Starting a session | `/start` |
| Ending a session | `/end` |
| Creating an implementation plan | `/plan` |
| Investigating a topic | `/research` |
| Making a strategic decision | `/c-suite` |
| Setting up a new project | `/init` |
