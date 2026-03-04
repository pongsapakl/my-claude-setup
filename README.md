# My Claude Team

[Claude when you update readme please dont edit this part. This is handwritten (or handtyped) and I wanted to keep it as is. Below ## Quickstart you can do whatever you are born to do.]

It appears to me that there are so many productivity frameworks out there trying to utilize the function of agents, skills, etc. as provided by Claude. Yet I find many of them are more generic and load everything up front (which I find somewhat annoying) and not tailored to my own needs. So, I created a custom one based on those frameworks out there and made it fit my requirements a little more. It is very basic yet so powerful for my workflow.

This repo serves as my archive/backup for my workflow (integrated with Claude Code, obviously), yet I try to make it more generic by moving project-specific information to the `CLAUDE.md` file and letting agents read from there to make this tool more reusable for other projects. If you find this interesting, please feel free to test it out, fork it—comments are appreciated, yet I can't confirm I'll fix issues since it is my workflow for my needs after all. Also feel free to basically fetch this to Claude and let it generate your own version of this. Well, maybe fetching some other repo might provide more polished ideas, but whatever.

[UPDATE v.0.5.0] I also feels like handing over session for me is really important. Despite there are things like `/compact` it is still not as good as i think it can be. That is why I use a file-based system to hand over session. This way, not only agent, but also us humans can catch up on what is left, what to do next more easily. Key is what is just done, and what to do next (along with related file). 

## Quick Install

```bash
/plugin marketplace add pongsapakl/my-claude-team 
/plugin install my-claude-team 
```

## Functionality

There are 3 main parts: agents, skills, and rules.

The most useful thing is **skills** here (8 total):

### `/init`

Bootstrap a new project workspace with the docs structure and WORK.md. Run this once when starting a new project.

```
/init
```

- Creates `docs/sessions/`, `docs/decisions/`, `docs/research/`, `docs/plans/`
- Creates `WORK.md` (live state file) from template
- Asks per-folder which to add to `.gitignore`
- Smart merge if structure already exists
- Offers to migrate legacy `.claude/memory/session-logs/`

### `/start`

Session onboarding — reads current state and gets you up to speed in under 30 seconds.

```
/start
```

- Reads `WORK.md` and latest session log from `docs/sessions/`
- Validates handoff quality ("Next Session Starts With" field)
- Presents onboarding summary (phase, what was done, handoff note, blockers)
- Flags incomplete handoffs
- Asks what to focus on

### `/end`

Session close with quality-tested handover. This replaced the old `/session-log` skill — it does everything session-log did, plus creates an actionable bridge to the next session.

```
/end
```

- Scans conversation for accomplishments, decisions, and open items
- Confirms summary with user before writing
- Writes session log to `docs/sessions/YYYY-MM-DD-topic.md`
- Quality-tests "Next Session Starts With" (must be verb-starting, specific, self-contained, and half-done-aware)
- Updates `WORK.md` with current state
- Optionally writes ADRs to `docs/decisions/` and research notes to `docs/research/`

### `/c-suite [question]`

This is where I find it a little [chunibyo](https://en.wikipedia.org/wiki/Ch%C5%ABniby%C5%8D) when you say "let's call a C-level meeting!" Yet, I find it so useful to have a round-robin debate with several subagents. It actually refines your ideas from several perspectives (though each C-persona might need more time to develop). I call it when I want to get the big visionary direction of the project, that kind of thing. You can do something like:

```
/c-suite Should we position our v3.0 as a premium version or a free version with upgradeable features?
```

- Each agent (CEO, CTO, CMO, CFO, Product Lead) provides their perspective
- You respond after each agent—can add your verdict, rationale, or random thoughts there
- Say "synthesize" to get final recommendation
- Decision saved to `docs/decisions/`

### `/research [topic]`

Intelligent research skill that auto-detects scope and calls relevant C-level agents only when needed for big decisions.

```
/research can we do real-time audio in browser?
```

**Quick Mode** (5-15 min):
- Single focused question, 1-3 searches
- Present facts in digestible format
- No agents called, user decides

**Deep Mode** (30-60 min):
- Multiple options, 5+ searches
- Auto-invokes relevant C-level agents for verdict
  - Architecture → CTO, Product Lead
  - Strategy → CEO, Product Lead, CMO
  - Cost → CFO, CTO
  - Security → Security Officer, CTO
- Comprehensive analysis with citations
- Synthesized recommendation

**Meta-Step** (when unclear):
- Presents quick scope assessment
- Asks: Quick or Deep research?
- Clear format for easy decision

### `/plan [feature]`

Creates implementation plans that live in `docs/plans/` and are referenced from `WORK.md`. Plans work across sessions — when Claude's context is full or credits run out, you can still use the plan to continue later.

```
/plan step-by-step adding authentication feature on v3.0
```

- Asks clarifying questions (purpose, rationale, timeline)
- Creates rationale-driven plan with task groups
- Exports to `docs/plans/YYYY-MM-DD-topic.md`
- Updates `WORK.md` Active Plan field

**Update**: Looks like Anthropic has this [Clear Context before Plan Exeuction](https://x.com/bcherny/status/2012663636465254662) integrated, they are targetting similar pain point so this skill might be deprecated in the future soon. Though I still find it useful to have a log of what is discussed and what is done outside claude folder, and this feature still doesn't solve cross session / cross model integrity cleanly (i guess they will be able to sooner or later). [I sometime use claude to plan and antigravity to execute to save cost lol]

### `/deployment-check [--full]`

I'm too dumb to do security checks on my own when deploying, so I let it do it for me. Now handles monorepo (frontend/backend split), API contract validation, and license compliance.

```
/deployment-check              # Standard (Security + Code Review)
/deployment-check --full       # Add License Compliance check
```

**What it checks**:
- Security Officer: secrets, HTTPS, vulnerabilities, API contracts, GCP/Vercel config (BLOCKING)
- Code Reviewer: TypeScript, SEO, accessibility, performance, monorepo structure (WARNINGS)
- License Officer (--full only): GPL/AGPL violations, JUCE license, attribution files (ADVISORY)

Unified report: BLOCKED / APPROVED WITH WARNINGS / APPROVED

### `/infra-check`

Config drift detection—compares actual GCP/Vercel settings vs what's documented in CLAUDE.md. Useful for quarterly audits or when onboarding.

```
/infra-check
```

- Checks: Cloud Run memory/regions, Vercel env vars, GitHub Actions workflows
- Reports drift with fix options (update docs or revert config)

---
## Other Components

### Agents
All 8 agents are used in the skills above—I don't normally use them standalone, but it's possible to activate them individually if needed.

| Agent | Focus | When They Help | Impact |
|-------|-------|----------------|---------|
| **CEO** | Strategy, market fit | Business decisions, positioning | Advisory |
| **CTO** | Technical architecture | Tech stack, feasibility, scalability | Advisory |
| **CMO** | Marketing, messaging | Landing pages, user acquisition | Advisory |
| **CFO** | Financial, ROI | Costs, pricing, resources | Advisory |
| **Product Lead** | UX, features | User experience, prioritization | Advisory |
| **Security Officer** | Security | Deployment safety, vulnerabilities, API contracts | **BLOCKING** |
| **Code Reviewer** | Quality, practices | Code review, performance, SEO, monorepo | Warnings |
| **License Officer** | Licensing | GPL violations, commercial licenses, attribution | Advisory |

### Rules
Just a collection of rules that I find useful to have in the workflow, not really critical (tbh I don't get the 'rules' function, but CC recommended me to add it there so why not haha).


## Personal Project Information

Since the agents and some workflows will look for your information, it is useful to add it to the `CLAUDE.md` file. Some example information can be as below. Feel free to add and build whatever feels right for you.

```markdown
## Agent Context (for Virtual C-Suite Team)

### Company Information
- **Company Name**: MyStartup
- **Stage**: MVP validation

### User Profile
- **Background**: Full-stack developer
- **Expertise**: Backend (Node.js), learning React

### Current Projects
- **Main App** (`app/`): Next.js, TypeScript - SaaS product - MVP in development
```
