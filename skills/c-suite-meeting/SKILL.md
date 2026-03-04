---
name: c-suite-meeting
description: Orchestrate round-robin debates between C-Suite agents (CEO, CTO, CMO, CFO, Product Lead) when user asks strategic questions like "should we", feature decisions, or business/product discussions. Auto-invokes for decision-making scenarios.
allowed-tools: [Task, AskUserQuestion, Write, Read]
---

# C-Suite Meeting - Round-Robin Debate Orchestration

This skill orchestrates multi-agent discussions with user participation between each agent's input.

## When to Auto-Invoke

Trigger when user asks:
- "Should we..." (strategic decisions)
- "Which is better, X or Y?" (comparing options)
- Feature/product decisions
- Tech stack choices
- Pricing/positioning questions
- Any question requiring multiple perspectives

## How Round-Robin Debates Work

### Flexible Flow (User Controls)

1. **Round 1**: Each agent presents initial perspective
   - CEO → [PAUSE for user input]
   - CTO → [PAUSE for user input]
   - CMO → [PAUSE for user input]
   - CFO → [PAUSE for user input]
   - Product Lead → [PAUSE for user input]

2. **Round 2+**: Agents respond to user feedback and each other
   - Continue until user says "synthesize", "decide", or convergence is reached
   - User can ask specific agents to respond
   - User can provide direction at any point

3. **Convergence**: Synthesize final recommendation
   - Highlight agreements
   - Note remaining disagreements
   - Present recommendation with trade-offs
   - Ask user for final decision

## Orchestration Instructions

As the orchestrator of C-Suite meetings:

### Step 1: Understand the Question

- Parse user's question to identify the decision being made
- Identify which perspectives are most relevant
- Note any constraints mentioned by user

### Step 2: Invoke Agents in Sequence

**For each agent**:

Use the Task tool to spawn the agent subagent with context:

```
Prompt template:
"You are participating in a C-Suite meeting about: [TOPIC]

User's question: [USER_QUESTION]

Context from previous agents:
[If Round 2+, include what other agents said and user's responses]

Provide your perspective as [CEO/CTO/CMO/CFO/Product Lead].
Focus on [strategic/technical/marketing/financial/UX] considerations.
Be concise but thorough (2-4 paragraphs).
Address any concerns raised by user or other agents if this is Round 2+."
```

**After each agent responds**:

1. Present their response to user
2. Use AskUserQuestion to pause and get user input:
   - "What's your take on [AGENT]'s perspective?"
   - Options: ["I agree", "I have concerns", "I have questions", "Continue to next agent"]
3. Record user's response for next round's context

### Step 3: Detect Convergence

After all agents have spoken at least once, check for:

- **User signals**: Says "synthesize", "decide", "make recommendation"
- **Agent alignment**: All agents agreeing on same solution
- **Diminishing returns**: Agents starting to repeat same points
- **User decision**: User has made up their mind

### Step 4: Synthesize & Recommend

When convergence detected or user requests:

1. **Summarize key points**:
   - What each agent recommended
   - Where they agreed
   - Where they disagreed

2. **Present trade-offs**:
   - Pros/cons of leading options
   - Risks and opportunities

3. **Make recommendation**:
   - "Based on the discussion, the C-Suite recommends: [CHOICE]"
   - "Rationale: [WHY]"
   - "Key trade-offs accepted: [WHAT WE'RE GIVING UP]"

4. **Ask for final decision**:
   - "Do you approve this recommendation?"
   - "Would you like another round of debate on any aspect?"

### Step 5: Document Decision

Once user decides:

1. Save decision record to `docs/decisions/YYYY-MM-DD-topic.md`
2. Use the template format:
   ```markdown
   # Decision Record: C-Suite Meeting — [TOPIC] - [DATE]

   ## Question
   [User's original question]

   ## Participants
   - CEO
   - CTO
   - CMO
   - CFO
   - Product Lead
   - User (You)

   ## Discussion Summary

   ### Round 1
   **CEO**: [Summary of position]
   **User Response**: [What user said]

   **CTO**: [Summary of position]
   **User Response**: [What user said]

   [etc...]

   ### Round 2
   [If applicable]

   ## Key Points
   - [Point 1]
   - [Point 2]
   - [Point 3]

   ## Disagreements
   - [Agent X] vs [Agent Y] on [topic]: [Summary]

   ## Final Decision
   **Decision**: [What was decided]
   **Rationale**: [Why]
   **Trade-offs Accepted**: [What we're giving up]

   ## Action Items
   - [ ] [Action 1]
   - [ ] [Action 2]
   ```

### Step 6: Additional Artifacts (Optional)

If the meeting produced artifacts beyond the decision:
- **Research findings** from agent analysis → offer to save to `docs/research/YYYY-MM-DD-topic.md`
- **Implementation plan** from the decision → offer to save to `docs/plans/YYYY-MM-DD-topic.md`

These are opt-in — ask the user if they want these additional files.

## Example Flow

### User Question
"Should we show pricing on the landing page or have a 'Contact Sales' button?"

### Round 1

**Invoke CEO agent**:
- CEO provides strategic perspective
- User responds: "Good point about transparency"

**Invoke CTO agent**:
- CTO provides technical ease assessment
- User responds: "Either is fine technically, got it"

**Invoke CMO agent**:
- CMO provides marketing/conversion perspective
- User responds: "Interesting data on conversion rates"

**Invoke CFO agent**:
- CFO provides financial/pricing flexibility angle
- User responds: "Makes sense for MVP uncertainty"

**Invoke Product Lead**:
- Product provides UX perspective
- User responds: "I like the MVP simplicity argument"

### Check for Convergence

User says: "I think I'm leaning toward Contact Sales based on what everyone said"

### Synthesize

"Based on the C-Suite discussion:

**Consensus**: Contact Sales button for MVP
**Supporting arguments**:
- CEO: Maintains competitive flexibility
- CMO: Allows lead qualification and personalization
- CFO: Lets us test pricing through conversations before committing
- Product Lead: Simpler MVP, can add pricing table v2
- CTO: No technical preference

**Trade-off**: Lower conversion rate vs pricing flexibility
**Recommendation**: Contact Sales for MVP, add pricing table after 10-20 customer conversations (per CFO suggestion)

Do you approve this recommendation?"

User: "Yes, approved"

### Document

Create session log with full transcript and decision.

## Important Guidelines

1. **User always decides**: Agents advise, user chooses
2. **Pause frequently**: User input after EVERY agent in Round 1
3. **No forced rounds**: User controls when to synthesize
4. **Context is key**: Pass previous responses to agents in Round 2+
5. **Document everything**: Session log is source of truth

## Handling Edge Cases

### User asks specific agent to respond
- Invoke only that agent with full context
- Continue normal flow afterward

### User wants to skip an agent
- Skip and continue to next agent
- Note in session log which agents participated

### Debate goes in circles
- Gently suggest synthesis: "We've heard all perspectives. Would you like me to synthesize recommendations?"
- If user declines, continue debate

### User changes question mid-discussion
- Acknowledge pivot
- Start new round with revised question
- Can reference previous discussion if relevant

## Success Criteria

- User feels heard and participates actively
- All relevant perspectives presented
- Decision is well-reasoned and documented
- User understands trade-offs
- Session log captures full context for future reference

## Post-Meeting

After documenting decision:
1. Decision record saved to `docs/decisions/`
2. The `/end` skill will reference this decision when closing the session
3. Decision is searchable in `docs/decisions/` for future reference

This completes the C-Suite meeting cycle.
