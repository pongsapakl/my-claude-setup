End the current session and create handover.

Use the end skill to close the session with a quality-tested handoff:

**The skill will**:
1. Scan conversation for accomplishments, decisions, and open items
2. Confirm summary with user before writing
3. Write session log to `docs/sessions/YYYY-MM-DD-topic.md`
4. Quality-test "Next Session Starts With" (4 criteria: verb-starting, specific, self-contained, half-done-aware)
5. Update `WORK.md` with current state
6. Optionally write ADRs to `docs/decisions/` and research notes to `docs/research/`

**Output**: Session log + updated WORK.md + quality-tested handoff note.
