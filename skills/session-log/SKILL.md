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

**Enhanced process with multi-modal enforcement:**

1. **Discover artifacts** (Step 0) - Use `git log`, `find` to gather objective evidence of what happened
2. **Detect scope** (Step 1) - Classify as SMALL/MEDIUM/LARGE session
3. **Multi-pass outline** (Step 2, if LARGE) - Create chronological phase outline, then document each phase
4. **Analyze conversation** - Review full conversation history (systematically via outline for LARGE sessions)
5. **Extract key info** - Summary, decisions, technical details, challenges, action items (phase-by-phase for LARGE)
6. **Self-verify** (Step 3) - Explicit checklist verification before saving
7. **Generate filename** - `YYYY-MM-DD-brief-topic.md` based on main topic
8. **Save log** - Write to `{cwd}/.claude/memory/session-logs/` (project-local, not global)
9. **Confirm** - Show verification checklist and summary to user

**No menus, no questions** - fully automatic based on context and artifacts.

**IMPORTANT**: Always save to the current working directory's `.claude/memory/session-logs/`, NOT to `~/.claude/`. Each project maintains its own session logs.

---

## Step 0: Discover Session Artifacts (REQUIRED - Do This FIRST)

Before analyzing conversation, gather objective evidence of what happened:

**Required checks (use these tools)**:

1. **Check for plan files**:
   ```bash
   find . -name "*-PLAN.md" -o -name "*.plan.md" -type f | grep -v node_modules
   ```
   - If found: Read it to understand original intent
   - Compare planned work vs. actual work completed

2. **Check git history**:
   ```bash
   git log --oneline --since="6 hours ago" --no-merges
   git diff HEAD~10..HEAD --stat
   ```
   - What commits were made during this session?
   - What files were actually changed?
   - Commit messages reveal phases of work

3. **Check for prior session logs**:
   ```bash
   ls -lt .claude/memory/session-logs/ | head -10
   ```
   - Is this continuing previous work?
   - Should link to related sessions

4. **Check current directory context**:
   ```bash
   pwd
   ls -la
   ```
   - Understand project structure

**Why this matters**: These artifacts provide OBJECTIVE evidence of what happened, preventing recency bias. Can't ignore what git commits and plan files show.

**Store findings**: Keep mental note of:
- Plan file path (if exists)
- Number of commits (indicates scope)
- Files modified (indicates breadth)
- Related sessions (for linking)

---

## Step 1: Detect Session Scope (Auto-Select Process)

Analyze session to determine complexity and select appropriate documentation process.

**Classification criteria**:

**SMALL Session**:
- < 20 conversation turns
- Single focused task
- 0-2 git commits
- No plan file
- Example: "Fixed typo in README"
- **Process**: Standard brief template

**MEDIUM Session**:
- 20-50 conversation turns
- Multiple related tasks
- 2-5 git commits
- May have informal planning
- Example: "Implemented user profile feature"
- **Process**: Standard comprehensive template

**LARGE Session** (triggers multi-pass):
- 50+ conversation turns, OR
- Plan file exists in project, OR
- 5+ git commits, OR
- Multiple distinct phases evident, OR
- Session spans 2+ hours
- Example: "Built payment system, debugged integration, refactored"
- **Process**: **MANDATORY Multi-Pass Documentation** (Step 2)

**How to detect**:
- Count turns in conversation
- Check if plan file found in Step 0
- Count git commits from Step 0
- Scan conversation for phase markers ("Now let's...", "Next we'll...", "I'm getting an error...")

**Decision**: If ANY criterion for LARGE is met → proceed to Step 2 (Multi-Pass)
Otherwise → skip to Step 3 (Select Template)

---

## Step 2: Multi-Pass Documentation (LARGE Sessions ONLY)

**This process is MANDATORY for LARGE sessions. Do not skip.**

### PASS 1: Create Chronological Outline

**Objective**: Map the ENTIRE session journey before documenting details.

**How to execute**:

1. **Scroll to conversation beginning** (not recent messages - go to turn 1)

2. **Identify phase transitions** - Look for markers:
   - Initial request/planning: "Let's start by...", "I need to...", "Can you help me..."
   - Implementation start: "Now let's build...", "First, we'll..."
   - Debugging: "I'm getting an error...", "This isn't working...", "Let's fix..."
   - Testing: "Let's test...", "Try running..."
   - Refinement: "Now let's improve...", "Add...", "Refactor..."

3. **Cross-reference with artifacts**:
   - Git commits show phase boundaries (implementation → debugging → refinement)
   - Plan file shows intended phases vs. actual
   - File changes indicate scope of each phase

4. **Create timeline outline**:
   ```
   Phase Outline:
   1. [Turn 1-15 / Commits: none] Planning: Discussed architecture for payment system
      - Referenced: PAYMENT-PLAN.md
      - Decision: Use Stripe, PostgreSQL for transactions

   2. [Turn 16-35 / Commits: a1b2c3, d4e5f6] Implementation: Built Stripe integration
      - Files: payment.ts, stripe-client.ts, schema.sql
      - Added webhook endpoint, database tables

   3. [Turn 36-60 / Commits: f7g8h9] Debugging: Fixed webhook signature validation
      - Issue: Signature mismatch errors
      - Root cause: Timestamp validation window too narrow
      - Solution: Increased tolerance to 5 minutes

   4. [Turn 61-75 / Commits: i0j1k2] Testing: Manual testing and edge cases
      - Tested successful payment flow
      - Added failed payment handling

   5. [Turn 76-85 / Commits: l3m4n5] Refinement: Error handling and logging
      - Added structured logging
      - Improved error messages
   ```

5. **Verify completeness**:
   - Does outline cover conversation start to end?
   - Are all git commits accounted for?
   - If plan file exists, are all planned items mentioned?
   - If NO → scroll back further and find missing phases

### PASS 2: Document Each Phase

**Objective**: Using the outline, document each phase comprehensively.

For each phase in outline:
- **Goal**: What was this phase trying to achieve?
- **Work done**: Specific accomplishments (reference commits, files)
- **Decisions**: What choices were made and why?
- **Challenges**: What problems arose?
- **Outcome**: What was achieved?

**This two-pass approach forces comprehensive coverage - you can't skip phases once they're outlined.**

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

### Implementation Session (LARGE - Multi-Phase)

Use when session is LARGE (50+ turns, has plan, or multiple phases):

```markdown
# Implementation Session: {Topic} - {Date}

> **Session Type**: LARGE Multi-Phase
> **Duration**: {X hours or Y turns}
> **Commits**: {X commits}
> **Plan Reference**: {Link to plan file if exists}

## Session Overview
{1-2 sentence summary of entire session from start to finish}

## Initial Objective
{What was the original goal? Quote from plan file or early conversation}

---

## Phase Breakdown

### Phase 1: {Phase Name} (Turns X-Y / Time: HH:MM)

**Goal**: {What was this phase trying to achieve?}

**Work Completed**:
- {Specific accomplishment 1}
- {Specific accomplishment 2}

**Key Decisions**:
- **{Decision}**: {What was decided and why}

**Technical Details**:
- Files: {list}
- Commits: {hashes if applicable}
- Key changes: {summary}

**Challenges**: {If any - what and how resolved}

---

### Phase 2: {Phase Name} (Turns X-Y / Time: HH:MM)
{Repeat above structure}

---

### Phase 3: {Phase Name} (Turns X-Y / Time: HH:MM)
{Repeat above structure}

---

{Continue for all phases identified in outline}

---

## Technical Summary

**Files Changed** (across all phases):
- `path/to/file1.ts`: {overall changes}
- `path/to/file2.ts`: {overall changes}

**Git Commits**:
```
a1b2c3 - Initial implementation
d4e5f6 - Add webhook handling
f7g8h9 - Fix signature validation
```

**Key Architectural Decisions** (across all phases):
1. **{Decision}**: {What was decided}
   - Rationale: {Why}
   - Alternatives considered: {Other options}
   - Phase decided: {When}
   - Trade-offs: {What we're giving up}

## Evolution & Learning

**How work evolved**: {Describe the journey from start to finish - what changed, what was learned}

**Initial approach vs. final**: {What was different? Why?}

## Challenges Encountered

- **{Challenge 1}** (Phase X): {Description}
  - Resolution: {How solved}
  - Learning: {What was learned}

- **{Challenge 2}** (Phase Y): {Description}
  - Resolution: {How solved}

## Accomplishments

{What was ultimately achieved? Compare to initial objective. Quantify where possible.}

## Action Items

- [ ] {Next step 1}
- [ ] {Next step 2}

## Related Documents

- Plan: {Link to plan file}
- Prior sessions: {Links to related session logs}
- Git range: {commit range}

## Next Session Pickup Point

{Where did we leave off? What context does next session need? What should be tackled next?}
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

---

## Step 3: Self-Verification (REQUIRED Before Saving)

**Before saving the log, explicitly verify completeness.**

**Output this checklist in your response to user:**

```
📋 Session Log Self-Verification:

✓ Artifact Review:
  [✓/✗] Checked for plan files: {YES - found X.md / NO - none found}
  [✓/✗] Reviewed git history: {YES - X commits / NO - no changes}
  [✓/✗] Linked related sessions: {YES - X sessions / NO - none found}

✓ Comprehensiveness Check:
  [✓/✗] Identified all phases: {List: Planning, Implementation, Debugging, etc.}
  [✓/✗] Initial goal captured: {YES - "quote from conversation" / NO}
  [✓/✗] Implementation approach documented: {YES / NO}
  [✓/✗] Challenges documented: {YES - X challenges / NO challenges}
  [✓/✗] Full conversation reviewed: {YES - scrolled to turn 1 / Used outline}

✓ Quality Check:
  [✓/✗] Log tells complete story: {Can someone understand this 6 months later? YES/NO}
  [✓/✗] References external artifacts: {Plan files, commits, prior logs}
  [✓/✗] Evolution/progression clear: {Can trace journey from start to end? YES/NO}

✓ Template Selection:
  Session classified as: {SMALL / MEDIUM / LARGE}
  Template used: {Implementation / Research / General}
  Multi-pass outline: {YES - created outline / NO - not needed}
```

**CRITICAL**: If any [✗] appears, STOP and gather that information before saving.

**After verification passes, proceed to save the log.**

---

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

- **Bash**:
  - Get current working directory (`pwd`)
  - Get current date for filename (`date`)
  - Discover artifacts (`git log`, `git diff`, `find`)
  - List prior session logs (`ls`)
- **Read**:
  - Check existing sessions for related content
  - Read plan files (e.g., `*-PLAN.md`)
  - Review git log output
- **Write**: Save session log file to `{cwd}/.claude/memory/session-logs/`
- **Grep**: Search for related sessions or specific patterns (optional)
- **Glob**: Find plan files (`*.plan.md`, `*-PLAN.md`) and session logs

## Key Principles

1. **Automatic** - No user input required, analyze conversation context
2. **Comprehensive** - Capture all important decisions and technical details
3. **Artifact-Grounded** - Reference external evidence (git commits, plan files, prior sessions)
4. **Phase-Aware** - Identify and document distinct phases for complex sessions
5. **Scope-Adaptive** - Adjust detail level based on session complexity
6. **Self-Verified** - Explicit verification checklist before saving
7. **Searchable** - Use consistent format and clear filenames
8. **Related** - Link to other relevant session logs when applicable
9. **Actionable** - Always include next steps/action items

---

Remember: Analyze the conversation thoroughly and create a complete record. The user should be able to understand what happened just by reading the log, even months later.
