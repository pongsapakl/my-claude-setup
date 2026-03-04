---
name: plan
description: Create implementation plans optimized for cross-session continuity. Auto-invokes when user says "/plan", "create a plan", "create an implementation plan", "help me plan this out", "break this down into a plan".
allowed-tools: [Read, Grep, Glob, Bash, Write, AskUserQuestion]
---

# Plan Skill - Cross-Session Planning

Automatically creates implementation plans optimized for cross-session continuity, enabling any Claude session to pick up work without conversation history.

## When to Auto-Invoke

Trigger when user says:
- "/plan" command
- "create a plan" / "create an implementation plan"
- "help me plan this out" / "plan this out"
- "break this down into a plan"
- "make a plan for..."
- "update the plan" / "update the {project} plan"

**Pattern**: Auto-invoke on explicit planning requests

## How It Works

### Step 0: Assess Scope & Select Mode (Crucial)

**Determine the complexity level to select the right approach:**

**MODE A: LIGHTWEIGHT (Bug fixes, simple refactors, loose tasks)**
- **Triggers**: "Fix this bug", "Rename this file", "Move these folder", "Update config"
- **Characteristics**: Low risk, local impact, clear "correct" state.
- **Context Handling**: 
  - ❌ Skip session logs.
  - ✅ Quick file tree or grep.
  - ✅ **ASK FIRST**: Clarify intent before reading any code.
- **Output**: `docs/plans/YYYY-MM-DD-topic.md` (Checklist style)

**MODE B: COMPREHENSIVE (New features, complex logic, architecture)**
- **Triggers**: "Add new feature", "Design this system", "Refactor core logic"
- **Characteristics**: High risk, systemic impact, multiple valid solutions.
- **Context Handling**:
  - ✅ Read session logs (decisions).
  - ✅ Search for patterns.
  - ✅ Check dependencies.
- **Output**: `docs/plans/YYYY-MM-DD-topic.md` (Design doc style)

### Step 1: Understand Requirements (1-2 min)
- Parse user's goal or feature request
- Identify project context from user's request or workspace
- Determine if this is:
  - New plan creation
  - Updating existing plan (detect from "update" keyword)
  - Plan for new feature, full project, or phase

**Tools**: Grep (search for existing plans), Glob (find project files)

### Step 2: Gather Context (CONDITIONAL - depends on Step 0)

**For SMALL tasks (refactors, reorganizations)**:
- Quick directory structure check (tree/find command)
- Search for existing plans only (no session logs)
- SKIP to Step 3 immediately - ask questions first!

**For LARGE tasks (new features, integrations)**:
- Search `docs/sessions/` for related session logs and `docs/decisions/` for architectural decisions
- Read relevant existing plans in `docs/plans/` if updating
- Identify similar patterns in codebase
- Check for dependencies or blockers

**Tools**: Read (minimal for small tasks), Grep (search patterns), Glob (find files), Bash (tree/find)

### Step 3: Clarify Goals & Rationale (CRITICAL)
**Use AskUserQuestion extensively** - better to ask 5 questions upfront than build the wrong thing.

**Essential questions**:
1. **Purpose/Value**: "What problem does this solve? What value does it provide?"
2. **Approach**: "Why this approach? Have you considered alternatives?"
3. **Success Metric**: "How will you know it worked? What's the success metric?"
4. **Constraints**: "Any constraints or non-goals? What should we NOT do?"
5. **Timeline**: "Timeline? (e.g., 1 week, 1 month, flexible)"
6. **Export location**: "Should I export to project directory?" (Default: Yes)

**Rationale**: Clarifying WHY upfront reduces risk of unnecessary work without good return.

**Tools**: AskUserQuestion

### Step 4: Structure Plan (Choose Template)

**Select the template matching your Step 0 assessment.**

**Option A: Lightweight Template** (Speed & Clarity)
- **Format**: Simple checklist.
- **Focus**: WHAT to do, verify it works.
- **Content**:
  - Brief Context (1-2 sentences)
  - Tasks (Checklist)
  - Verification (How to test)

**Option B: Comprehensive Template** (Robustness & Rationale)
- **Format**: Structured design document.
- **Focus**: WHY we are doing it, architectural fit.
- **Content**:
  - Goal & Rationale (Critical for future context)
  - Key Changes (Component level)
  - Dependencies/Risks
  - Detailed Implementation Steps
  - Verification Plan

**Tools**: Write (use appropriate template), Bash (get date)

### Step 5: Export & Confirm (1 min)
- Save full plan to `docs/plans/YYYY-MM-DD-topic.md` (single source of truth)
- Update `WORK.md` "Active Plan" field to point to the new plan file path
- Show summary with next steps

**IMPORTANT**: The exported plan is the authoritative version. Plans live in `docs/plans/` alongside other documentation artifacts.

**Tools**: Write, Read, Bash

## Plan Templates

### Option A: Lightweight Plan (Small Tasks)

```markdown
# {Task Name} - Implementation Plan (Lightweight)

> **Created**: {YYYY-MM-DD}
> **Type**: Lightweight

## Context
{Briefly: What are we fixing/changing and why?}

## Checklist
- [ ] {Action Item 1} <!-- file: path/to/file -->
- [ ] {Action Item 2}
- [ ] {Action Item 3}

## Verification
- [ ] {How to verify functionality}
```

### Option B: Comprehensive Plan (Large Tasks)

```markdown
# {Feature Name} - Implementation Plan

> **Created**: {YYYY-MM-DD}
> **Type**: Comprehensive

## Goals & Rationale
**Goal**: {What are we achieving?}
**Rationale**: {Why this approach? Trade-offs considered?}

## Key Changes
- **New Components**: {List new files/classes}
- **Modifications**: {List critical changes to existing files}

## Implementation Steps

### Phase 1: {Name} (Est: X hr)
**Goal**: {Specific objective of this phase}

- [ ] {Detailed Task 1}
  - **Details**: {Brief implementation note}
  - **File**: `{path}`
- [ ] {Detailed Task 2}

## Verification Plan
### Automated Tests
- {Command to run}

### Manual Verification
- {Step-by-step check}
```

## Example Workflow

### Creating New Plan (LARGE TASK - New Feature)

```
User: "/plan add voice input to [project-name]"
  ↓
Skill auto-invokes
  ↓
Step 0: Assess scope → LARGE (new feature, complex implementation)
  ↓
Step 1-2: Gathers context (searches for project plans, session logs, code)
  ↓
Step 3: Asks clarifying questions:
  - "Why voice input? What problem does it solve?"
  - "Have you considered privacy implications?"
  - "Success metric: accuracy threshold?"
  - "Timeline: 1 week or flexible?"
  ↓
Step 4: Structures plan with task groups + rationale
  ↓
Step 5: Exports to docs/plans/YYYY-MM-DD-voice-input.md + updates WORK.md Active Plan
  ↓
Confirms: ✅ Plan created with summary
```

### Creating New Plan (SMALL TASK - Refactoring)

```
User: "/plan reorganize [project-name]/Source/ directory structure"
  ↓
Skill auto-invokes
  ↓
Step 0: Assess scope → SMALL (file reorganization, structural change)
  ↓
Step 1: Quick tree command to see current structure
  ↓
Step 2: SKIP deep context (don't read session logs!)
  ↓
Step 3: Ask questions IMMEDIATELY:
  - "What logical components should be grouped together?"
  - "How do you envision the target structure?"
  - "Should we preserve git history with git mv?"
  - "What's the reasoning behind this reorganization?"
  ↓
Step 4: Structures plan based on user's vision
  ↓
Step 5: Exports to docs/plans/YYYY-MM-DD-source-reorganization.md + updates WORK.md Active Plan
  ↓
Confirms: ✅ Plan created with summary
```

### Updating Existing Plan

```
User: "update the [project-name] plan with new priorities"
  ↓
Skill detects "update" keyword
  ↓
Searches for existing plan files (Glob: "docs/plans/*.md")
  ↓
Reads current plan (Read tool)
  ↓
Asks: "What would you like to update?"
  1. Mark tasks complete
  2. Add new task group
  3. Revise timeline
  4. Add blocker
  5. Other
  ↓
User responds: "Add new task group for preset system"
  ↓
Updates plan preserving existing content
  ↓
Confirms: ✅ Plan updated with changes summary
```

## File Naming

### Plan Files
Format: `docs/plans/YYYY-MM-DD-kebab-case-topic.md`

Examples:
- `docs/plans/2026-03-04-user-authentication.md`
- `docs/plans/2026-03-04-api-refactor.md`

**Single source of truth**: Plans live in `docs/plans/`. The active plan is referenced in `WORK.md`.
## Detecting Plan Updates

## Tools Usage

- **Read**: Read existing plans in `docs/plans/`, session logs in `docs/sessions/`, code files for context
- **Grep**: Search for related patterns, architectural decisions in `docs/decisions/`
- **Glob**: Find files matching patterns (e.g., `docs/plans/*.md`)
- **Bash**: Get current date for metadata
- **Write**: Export plan to `docs/plans/`, update WORK.md Active Plan field
- **AskUserQuestion**: Clarify goals, rationale, constraints upfront

## Key Principles

1. **Assess Scope First** - Determine small vs large task to avoid wasting resources
2. **Ask Questions Early** - For small tasks, ask questions BEFORE gathering context (not after)
3. **Rationale-Driven** - Every task group explains WHY, not just WHAT
4. **Cross-Session Continuity** - Any agent can pick up work without conversation history
5. **File-Level Granularity** - Specify exact files to create/modify
6. **Mid-Level Detail** - Tasks at component level (1-3 hours), not function-level
7. **Minimal Code** - Pseudocode and interfaces only, no full implementations
8. **Essential Context** - Link to session logs, architectural decisions (only for large tasks)
9. **Natural Language** - No complex flags, detect intent from user's words

## Anti-Patterns to Avoid

**❌ DON'T: Over-research small tasks**
- Reading 400+ line session logs for simple file reorganization
- Deep-diving into code when only structure matters
- Gathering context before understanding user's vision

**✅ DO: Light touch for refactors**
- Quick `tree` command to see current structure
- Ask questions about desired end state
- Create plan based on user's vision, not historical context

**❌ DON'T: Make assumptions without asking**
- Analyzing structure and concluding "it's fine as-is"
- Proposing solutions before understanding the problem
- Assuming you understand domain relationships (e.g., NLP/Anchors/Interpolation)

**✅ DO: Ask clarifying questions first**
- "What logical components should be grouped?"
- "How do you envision the new structure?"
- "What's the reasoning behind this reorganization?"

**Remember**: For small tasks, user's vision > historical context. ASK FIRST, RESEARCH LATER (if needed).

## Quality Checklist

**Good plan has**:
- ✅ Clear, discrete tasks (each completable independently)
- ✅ File-level specificity (exact paths, what to create/modify)
- ✅ Realistic time estimates (based on similar past work)
- ✅ Essential context links (architectural decisions, related docs)
- ✅ Verification criteria (how to know it's done)
- ✅ Next session notes (pickup point for new sessions)
- ✅ Goal and rationale for each task group

**Bad plan has**:
- ❌ Vague tasks ("Implement NLP" - too broad)
- ❌ Full code implementations (defeats purpose of planning)
- ❌ No context links (new session has no background)
- ❌ Missing file paths (hard to locate what to change)
- ❌ No success criteria (unclear when task is done)
- ❌ Missing rationale (doesn't explain WHY)

## Success Metrics

**Plan is successful if**:
- User can pick up any task without reading full plan
- New session can continue work without conversation history
- Tasks are completable in 1-3 hour chunks
- Dependencies are clear (no wasted time on blocked tasks)
- Related context is linked (no searching for decisions)
- Rationale helps understand WHY we're building something

**Plan needs revision if**:
- User frequently asks "what does this mean?"
- Tasks are too large (taking 5+ hours each)
- Dependencies weren't clear (started blocked task)
- Missing critical context (had to search for decisions)
- Unclear purpose (why are we building this?)

---

Remember: **Better to ask 5 questions upfront than build the wrong thing.** Use AskUserQuestion extensively to clarify goals, rationale, and constraints before structuring the plan.
