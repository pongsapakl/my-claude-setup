# C-Suite Discussion Protocol

## How Round-Robin Debates Work

### Format
1. User asks a strategic question or raises a decision
2. C-Suite meeting skill orchestrates multi-agent discussion
3. Each agent (CEO, CTO, CMO, CFO, Product Lead) provides their perspective
4. **User participates** after each agent (not just at the end)
5. Rounds continue until user requests synthesis or convergence reached

### User Participation

**CRITICAL**: User must have opportunity to respond after EACH agent in Round 1
- Ask user for input: "What's your take on [AGENT]'s perspective?"
- Record user's response for context in subsequent rounds
- User can agree, disagree, ask questions, or provide constraints

### Round Structure

**Round 1: Initial Perspectives**
- CEO → User input → CTO → User input → CMO → User input → CFO → User input → Product Lead → User input

**Round 2+: Responses & Refinement**
- Agents respond to user feedback and address each other's concerns
- Continue until:
  - User says "synthesize" or "decide"
  - Clear convergence on a solution
  - Agents start repeating same points (diminishing returns)

**Convergence: Final Recommendation**
- Synthesize all perspectives
- Highlight agreements and disagreements
- Present recommendation with trade-offs
- Ask user for final approval

### Facilitation Guidelines

1. **Equal Voice**: All relevant agents must speak (unless user specifically excludes)
2. **User Control**: User decides when to move forward, skip, or synthesize
3. **Context Preservation**: Pass conversation history to agents in Round 2+
4. **Document Everything**: Full transcript goes into decision record

### Convergence Signals

Watch for:
- User explicitly requests synthesis
- All agents align on same recommendation
- User has made clear decision
- Discussion becoming circular

### Decision Documentation

After user approves decision:
1. Create decision record in `docs/decisions/YYYY-MM-DD-topic.md`
2. Include full transcript, key decisions, disagreements, action items
3. Decision records are stored in `docs/decisions/` and referenced by `/end`

## Tone & Style

- **Respectful Debate**: Agents can disagree but remain professional
- **Evidence-Based**: Reference examples, data, best practices when available
- **User-Centric**: Final decision is always user's, not agents'
- **Actionable**: End with clear next steps, not just discussion
