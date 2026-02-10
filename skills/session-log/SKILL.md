---
name: session-log
description: Saves detailed session logs based on conversation context. Auto-invokes when user asks to save/log session. (project)
allowed-tools: [Read, Write, Grep, Glob, Bash]
---

# Session Log - Automatic Session Documentation

Automatically creates detailed session logs based on conversation history.

## When to Auto-Invoke

Trigger when user says:
- "save this session" / "save the session"
- "log this session" / "log the session"
- "document this session"
- "/session-log" command

## How It Works

1. **Detect project directory** - Use `pwd` to get current working directory
2. **Analyze conversation** - Review full conversation history automatically
3. **Determine type** - Implementation (coding), Research (investigation), or General (planning/discussion)
4. **Extract key info** - Summary, decisions, technical details, research findings, challenges, action items
5. **Generate filename** - `YYYY-MM-DD-brief-topic.md` based on main topic
6. **Save log** - Write to `{cwd}/.claude/memory/session-logs/` (project-local, not global)
7. **Confirm** - Show summary to user

**No menus, no questions** - fully automatic based on context.

**IMPORTANT**: Always save to the current working directory's `.claude/memory/session-logs/`, NOT to `~/.claude/`. Each project maintains its own session logs.

---

## ⚠️ CRITICAL: Comprehensiveness Requirement

**ALWAYS capture the ENTIRE session, not just recent work.**

Session logs are diary-style execution history. They must enable understanding what happened months later.

**Before writing the log, verify:**
- ✅ Reviewed FULL conversation from start to finish
- ✅ Captured ALL phases of work (not just current/recent work)
- ✅ Included ALL context, decisions, and technical details
- ✅ Log tells complete story of what happened in session

**NEVER:**
- ❌ Create partial logs limited to "current work"
- ❌ Focus only on most recent activity
- ❌ Skip earlier phases or context

**If in doubt:** Scroll to the beginning of conversation and read forward. Capture everything.

---

## Session Log Templates

### Implementation Session

Use when session involved coding, technical work, or implementation:

```markdown
# Implementation Session: {Topic} - {Date}

## Objective
{What was the goal}

## Work Completed
- {Feature/fix/task 1}
- {Feature/fix/task 2}

## Technical Decisions
1. **{Decision}**: {What was decided}
   - Rationale: {Why}
   - Alternatives: {Other options considered}
   - Trade-offs: {What we're giving up}

## Code Changes
- Files modified: {list}
- Key changes: {summary}

## Challenges Encountered
- {Challenge}: {How resolved}

## Action Items
- [ ] {Next step 1}
- [ ] {Next step 2}

## Related Documents
- {Links to related files or sessions}

## Next Session
{What to tackle next}
```

### General Session

Use when session was planning, research, configuration, or discussion:

```markdown
# Session Log: {Topic} - {Date}

## Summary
{Brief overview of what happened}

## Decisions Made
1. {Decision 1}
   - Rationale: {Why}
2. {Decision 2}
   - Rationale: {Why}

## Discussion Points
- {Point 1}
- {Point 2}

## Action Items
- [ ] {Action 1}
- [ ] {Action 2}

## Related Documents
- {Links to related files or sessions}

## Next Steps
{What to do next}
```

### Research Session

Use when session involved web research, technical investigation, or feasibility analysis:

```markdown
# Research Session: {Topic} - {Date}

## Research Question
{What were we investigating?}

## Research Conducted

### Web Research
- {Search topic 1}: {Key finding}
- {Search topic 2}: {Key finding}
- {Search topic 3}: {Key finding}

### Technical Validation
- {Test/tool used}: {Result}
- {Local experiment}: {Outcome}

### Code/File Analysis
- {File examined}: {What we learned}

## Key Findings

### Finding 1: {Title}
**Source**: {URL or file path}
**Evidence**: {What was discovered}
**Implication**: {What this means for our decision}

### Finding 2: {Title}
**Source**: {URL or file path}
**Evidence**: {What was discovered}
**Implication**: {What this means for our decision}

## Options Analyzed

### Option A: {Name}
**Description**: {Brief overview}
**Pros**:
- {Benefit 1}
- {Benefit 2}
**Cons**:
- {Drawback 1}
- {Drawback 2}
**Feasibility**: {Technical verdict}
**Research Support**: {Citation numbers}

### Option B: {Name}
**Description**: {Brief overview}
**Pros**:
- {Benefit 1}
**Cons**:
- {Drawback 1}
**Feasibility**: {Technical verdict}
**Research Support**: {Citation numbers}

## Decision/Verdict
**Recommendation**: {What to do}
**Rationale**: {Why, based on research}
**Confidence Level**: {High/Medium/Low}
**Timestamp**: {When decided}
**Unknowns**: {What we still don't know}

## Citations
1. [{Title}]({URL}) - {Brief description}
2. [{Title}]({URL}) - {Brief description}
3. [{Title}]({URL}) - {Brief description}
{... all sources used in research}

## Action Items
- [ ] {Next step based on research}
- [ ] {Implementation task if decision made}

## Related Documents
- {Links to related session logs or files}
```

**When to use Research template**:
- Session has 3+ WebSearch/WebFetch calls
- Deep technical investigation or feasibility analysis
- Multiple options compared with citations
- Decision made based on research findings

## Confirmation Format

After saving, show user:

```
✅ Session log saved: {filename}

Key decisions: {count}
Action items: {count}
Related sessions: {count if any}
```

## Example Workflow

```
User: "save this session"
  ↓
Skill auto-invokes
  ↓
Gets working directory: pwd → /Users/user/Projects/MyProject
  ↓
Analyzes conversation (debugging SessionEnd hook)
  ↓
Determines: General session
  ↓
Generates filename: 2025-12-28-sessionend-hook-debugging.md
  ↓
Saves to /Users/user/Projects/MyProject/.claude/memory/session-logs/
  ↓
Confirms: ✅ Session log saved
```

## File Naming

Format: `YYYY-MM-DD-brief-topic-description.md`

Examples:
- `2025-12-28-sessionend-hook-debugging.md`
- `2025-12-18-project-deployment.md`
- `2025-12-15-api-refactor.md`

Use lowercase with dashes. Keep brief but descriptive (3-5 words max).

## Tools Usage

- **Bash**: Get current working directory (`pwd`) and current date for filename
- **Read**: Check existing sessions for related content
- **Write**: Save session log file to `{cwd}/.claude/memory/session-logs/`
- **Grep**: Search for related sessions (optional)

## Key Principles

1. **Automatic** - No user input required, analyze conversation context
2. **Comprehensive** - Capture all important decisions and technical details
3. **Searchable** - Use consistent format and clear filenames
4. **Related** - Link to other relevant session logs when applicable
5. **Actionable** - Always include next steps/action items

---

Remember: Analyze the conversation thoroughly and create a complete record. The user should be able to understand what happened just by reading the log, even months later.
