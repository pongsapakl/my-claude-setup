---
name: research
description: Conducts technical/market research and provides findings. Automatically calls relevant C-level agents for big decisions. (project)
allowed-tools: [WebSearch, WebFetch, Read, Grep, Glob, Bash, Task]
---

# Research - Intelligent Investigation & Analysis

Conducts research and automatically determines if C-level perspectives are needed.

## When to Auto-Invoke

Trigger when user says:
- "investigate {topic}"
- "research {topic}"
- "is it possible to {question}?"
- "can we do {question}?"
- "look into {topic}"
- "/research {topic}"

## Research Modes (Auto-Detected)

### Quick Research Mode (5-15 min)
**Characteristics**:
- Single focused question
- 1-3 web searches needed
- Clear technical answer exists
- User just needs facts to decide

**Example**: "Can pedalboard run on Vercel?"

**Process**:
1. Conduct focused research (1-3 searches)
2. Present findings in digestible format
3. No C-level agents invoked
4. User decides based on facts

**Output Format**:
```
## Quick Research: {Topic}

**Question**: {What we're investigating}

**Finding**: {Clear answer}

**Evidence**:
- {Source 1}: {Key fact}
- {Source 2}: {Key fact}

**Verdict**: {Yes/No/Depends + brief explanation}

**Your Decision**: {What user should consider}

Sources:
1. [{Title}]({URL})
2. [{Title}]({URL})
```

---

### Deep Research Mode (30-60+ min)
**Characteristics**:
- Multiple questions to answer
- 5+ web searches needed
- Architectural/strategic implications
- Affects product direction or significant effort

**Example**: "Can we do real-time audio in browser? Compare all options."

**Process**:
1. Conduct comprehensive research (5+ searches, technical tests)
2. Analyze multiple options with trade-offs
3. **AUTO-INVOKE relevant C-level agents** for verdict
4. Present integrated findings + agent perspectives
5. User makes final decision

**Which Agents to Call** (Auto-Select Based on Topic):

| Topic Type | Agents to Invoke | Why |
|------------|------------------|-----|
| **Architecture/Technology** | CTO, Product Lead | Technical feasibility + UX impact |
| **Product Strategy** | CEO, Product Lead, CMO | Market fit + positioning + messaging |
| **Cost/ROI** | CFO, CTO | Financial + technical effort |
| **Security/Compliance** | Security Officer, CTO | Risk + implementation |
| **Full Product Decision** | CEO, CTO, CMO, CFO, Product Lead | All perspectives |

**Output Format**:
```
## Deep Research: {Topic}

### Research Question
{What we're investigating}

### Key Findings

#### Finding 1: {Title}
**Evidence**: {What research showed}
**Source**: [{Title}]({URL})
**Implication**: {What this means}

#### Finding 2: {Title}
...

### Options Analyzed

#### Option A: {Name}
**Pros**: {List}
**Cons**: {List}
**Feasibility**: {Assessment}
**Cost**: {Rough estimate if applicable}

#### Option B: {Name}
...

---

### C-Level Perspectives

> **Note**: Called {Agent 1}, {Agent 2}, {Agent 3} based on decision type

**{Agent 1} (e.g., CTO)**:
{Technical perspective on options}

**{Agent 2} (e.g., Product Lead)**:
{UX/product perspective on options}

**{Agent 3} (e.g., CFO)** (if cost-relevant):
{Financial perspective on options}

---

### Recommendation
**Based on research + agent input**:
{Synthesized recommendation}

**Rationale**: {Why this makes sense}

**Your Call**: {What user needs to decide}

---

### Citations
1. [{Title}]({URL}) - {Brief description}
2. [{Title}]({URL}) - {Brief description}
...
```

---

### Meta-Step Mode (When Unclear)

**When scope is ambiguous**, present quick summary first and ASK:

```
## Research Scoping: {Topic}

**Your Question**: {What user asked}

**Initial Assessment**:
- Estimated searches needed: {X}
- Decision weight: {Low/Medium/High}
- Potential agents relevant: {List}

**How deep should I go?**

**Option 1: Quick Research** (5-15 min)
- Just answer the core question
- Present facts, you decide
- No agent perspectives

**Option 2: Deep Research** (30-60 min)
- Comprehensive analysis
- Multiple options compared
- C-level verdict from: {Agent list}

Which approach do you want?
```

**User responds**: "quick" or "deep" or just "go ahead" (defaults to deep if high-weight)

---

## Auto-Detection Logic

### Triggers for QUICK Mode:
- Question answerable with 1-3 sources
- "Can X do Y?" questions with clear yes/no
- Library/tool capability checks
- Syntax/API lookups

### Triggers for DEEP Mode:
- Words: "compare", "all options", "best approach", "architecture"
- Multiple "or" options: "X or Y or Z?"
- Strategic implications: "should we", "which product"
- Affects >1 week of work or product direction

### Triggers for META-STEP:
- User question is vague or open-ended
- Could be quick OR deep depending on interpretation
- Decision weight unclear from context

**Examples**:

| User Question | Mode | Reasoning |
|--------------|------|-----------|
| "Can pedalboard run on Vercel?" | Quick | Single yes/no, library check |
| "Investigate real-time audio options" | Deep | Multiple options, architecture |
| "Look into authentication" | Meta-Step | Unclear: Quick check or full strategy? |
| "Should we use Stripe or PayPal?" | Deep | Product decision, cost implications |
| "What's the ONNX Runtime API for loading models?" | Quick | Documentation lookup |

---

## C-Level Agent Selection (Deep Mode)

**Architecture/Technical Feasibility**:
```
Auto-invoke: CTO, Product Lead
Reason: Technical feasibility + UX impact
Example: "Can we do real-time audio in browser?"
```

**Product Strategy/Positioning**:
```
Auto-invoke: CEO, Product Lead, CMO
Reason: Market fit + user perception + messaging
Example: "Should we offer free tier or paid only?"
```

**Cost/Effort vs Value**:
```
Auto-invoke: CFO, CTO
Reason: ROI calculation + implementation effort
Example: "Is it worth building custom VST host?"
```

**Security/Risk**:
```
Auto-invoke: Security Officer, CTO
Reason: Risk assessment + mitigation approach
Example: "Should we store user audio files?"
```

**Major Product Decision**:
```
Auto-invoke: CEO, CTO, CMO, CFO, Product Lead
Reason: Full strategic alignment needed
Example: "Should we pivot to web-only product?"
```

---

## Agent Invocation Format

**Single Task call with all agents in parallel**:

```python
# Use Task tool once with multiple agent invocations
Task(
  subagent_type="my-claude-team:cto",
  prompt="{Research summary}\n\nProvide technical perspective on options A, B, C"
)

# Then invoke other relevant agents in same message
```

**Agent Output Format**:
```
{Agent Name}: {1-2 paragraph perspective}

Key points:
- {Insight 1}
- {Insight 2}
- {Recommendation}
```

**Synthesis**:
Combine agent perspectives with research findings into clear recommendation.

---

## Example Workflows

### Example 1: Quick Research
```
User: "Can ONNX Runtime run in browser?"

[Auto-detect: Quick mode - single capability check]

Research (2 searches):
- ONNX Runtime docs
- Browser support matrix

Output:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## Quick Research: ONNX Runtime Browser Support

**Question**: Can ONNX Runtime run in browser?

**Finding**: ✅ Yes, via ONNX Runtime Web (WASM)

**Evidence**:
- Official onnxruntime-web npm package
- Full browser support (Chrome, Firefox, Safari)
- Uses WebAssembly backend

**Verdict**: Fully supported for browser deployment

**Your Decision**: Safe to proceed with browser integration

Sources:
1. [ONNX Runtime Web Docs](https://...)
2. [Browser Compatibility](https://...)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[No agents called - straightforward fact]
```

---

### Example 2: Deep Research with Agents
```
User: "Investigate whether we should use Cloud Run or client-side WASM for audio processing"

[Auto-detect: Deep mode - architecture decision, multi-option, strategic]
[Auto-select agents: CTO, Product Lead, CFO]

Research (7 searches):
- Cloud latency benchmarks
- WASM audio performance
- Cost comparison
- User experience considerations

Output:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## Deep Research: Cloud Run vs Client-Side WASM

### Research Question
Which architecture for audio processing: server-side (Cloud Run) or client-side (WASM)?

### Key Findings

#### Finding 1: Latency Constraints
**Evidence**: Real-time audio requires <10ms, server-side is 100-350ms
**Source**: [Audio Latency Requirements](https://...)
**Implication**: Server-side cannot support real-time playback

#### Finding 2: Cost Structure
**Evidence**: Cloud Run = $0.40/hr active, WASM = free (user's CPU)
**Source**: [GCP Pricing](https://...)
**Implication**: WASM cheaper at scale

#### Finding 3: Complexity
**Evidence**: WASM requires compilation setup, Cloud Run uses existing binary
**Source**: [JUCE Emscripten Docs](https://...)
**Implication**: Cloud Run faster to ship

### Options Analyzed

#### Option A: Cloud Run
**Pros**:
- Use existing VST3 binary (no recompilation)
- Easier debugging
- Centralized processing

**Cons**:
- Cannot support real-time (<10ms latency)
- Ongoing costs ($10-15/month minimum)
- Network dependency

**Feasibility**: ✅ High (already have Docker setup)
**Cost**: ~$15/month for always-on instance

#### Option B: Client-Side WASM
**Pros**:
- Real-time capable (<10ms latency)
- Zero server costs
- Works offline

**Cons**:
- Need to compile JUCE to WASM (1-2 days setup)
- User's device does the work
- Harder to debug

**Feasibility**: ✅ High (JUCE has emscripten port)
**Cost**: Free (user's CPU)

---

### C-Level Perspectives

> **Called**: CTO, Product Lead, CFO (architecture + UX + cost decision)

**CTO**:
From a technical standpoint, both are feasible. Cloud Run is the fast path - we have the binary, it works, deploy in hours. WASM is the right path - better latency, no server costs, but needs 1-2 days to set up compilation toolchain.

Key consideration: Do we need real-time? If yes, WASM is the only option. If offline demo is fine, Cloud Run works.

**Product Lead**:
Real-time playback is crucial for serious evaluation. Users want to plug in their guitar and hear the effect immediately - that's the "wow" moment before buying the VST. An offline demo with 2-5s processing doesn't create the same urgency.

However, for casual exploration (landing page demo), offline is acceptable. We might need both: quick demo for marketing, real-time for serious users.

**CFO**:
Cloud Run at $15/month seems cheap, but remember we're pre-revenue. Every fixed cost matters. WASM's free operation is attractive. The 1-2 day setup investment pays off immediately - no monthly burn.

If we get 100 users/day on Cloud Run, we're looking at $50-100/month. WASM scales to infinity at $0.

---

### Recommendation

**Based on research + agent input**:

**Short-term (MVP)**: Cloud Run for landing page demo
- Ship in hours, validate NLP concept
- Acceptable for marketing/exploration use case
- Low risk, proven working

**Medium-term (Real Product)**: Client-Side WASM for real-time
- Invest 1-2 days in WASM compilation
- Unlocks true real-time experience
- Zero ongoing costs

**Rationale**: Two different use cases need two different solutions. Don't force one architecture to serve both.

**Your Call**:
1. Do you want to ship Cloud Run demo first (this weekend)?
2. Then invest in WASM for real-time page (next week)?

Or skip Cloud Run entirely and go straight to WASM?

---

### Citations
1. [Real-Time Audio Latency](https://...) - Professional threshold
2. [Cloud Run Pricing](https://...) - Cost analysis
3. [ONNX Runtime WASM](https://...) - Browser support
4. [JUCE Emscripten Port](https://...) - WASM feasibility
... (20+ total sources)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Agents auto-invoked for strategic decision]
```

---

### Example 3: Meta-Step (Unclear Scope)
```
User: "Look into authentication"

[Auto-detect: Unclear - could be quick check OR full strategy]

Output:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## Research Scoping: Authentication

**Your Question**: "Look into authentication"

**Initial Assessment**:
- Could be: Library check (quick) OR full auth strategy (deep)
- Estimated searches: 2-10 depending on scope
- Decision weight: Medium-High
- Potential agents: CTO (technical), Security Officer (risk), Product Lead (UX)

**How deep should I go?**

**Option 1: Quick Research** (5-15 min)
- Just find out what auth libraries are available
- Basic pros/cons of popular options
- You decide which to use

**Option 2: Deep Research** (30-60 min)
- Compare OAuth vs JWT vs passwordless vs email magic links
- Security implications, UX trade-offs, cost
- Get CTO + Security Officer + Product Lead perspectives
- Full recommendation

Which approach?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Wait for user: "quick" or "deep" or "just find libraries"]
```

---

## Tools Usage

- **WebSearch**: Primary research tool (search web for information)
- **WebFetch**: Fetch specific documentation/articles
- **Read/Grep/Glob**: Search codebase for existing implementation
- **Bash**: Run local tests to validate claims
- **Task**: Invoke C-level agents in parallel for deep research mode

---

## Research Output

After research completes, save findings to `docs/research/YYYY-MM-DD-topic.md`.

- **Quick Research**: Ask user if they want to save (small findings may not need a file).
- **Deep Research**: Always save automatically — too much context to lose.

**If the research led to a decision** (e.g., "use Option A over B"), also write an ADR to `docs/decisions/YYYY-MM-DD-topic.md`. Research captures *what was investigated*; the ADR captures *what was decided and why*. Both are valuable.

The `/end` skill will reference these notes when closing the session.

---

## Key Principles

1. **Intelligent Scope Detection** - Auto-detect quick vs deep, ask if unclear
2. **Selective Agent Invocation** - Only call relevant agents, not entire C-suite
3. **Digestible Format** - Clear structure, easy to scan, actionable
4. **Citation-Rich** - Every claim backed by source
5. **User Control** - Meta-step when scope unclear, user always decides final action
6. **Efficiency** - Quick mode for simple questions (don't over-research)
7. **Persistent Output** - Research saved to `docs/research/` for future reference

---

Remember: The goal is to give the user the right level of research for their question - not too shallow (missing key info), not too deep (wasting time on obvious questions).
