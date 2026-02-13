# My Claude Team

It appears to me that there are so many productivity frameworks out there trying to utilize the function of agents, skills, etc. as provided by Claude. Yet I find many of them are more generic and load everything up front (which I find somewhat annoying) and not tailored to my own needs. So, I created a custom one based on those frameworks out there and made it fit my requirements a little more. It is very basic yet so powerful for my workflow.

This repo serves as my archive/backup for my workflow (integrated with Claude Code, obviously), yet I try to make it more generic by moving project-specific information to the `CLAUDE.md` file and letting agents read from there to make this tool more reusable for other projects. If you find this interesting, please feel free to test it out, fork it—comments are appreciated, yet I can't confirm I'll fix issues since it is my workflow for my needs after all. Also feel free to basically fetch this to Claude and let it generate your own version of this. Well, maybe fetching some other repo might provide more polished ideas, but whatever.

## Quick Install

```bash
/plugin marketplace add pongsapakl/my-claude-team 
/plugin install my-claude-team 
```

## Functionality

There are 4 main parts: agents, skills, commands, and rules.

The most useful thing is **skills** here (6 total):

### `/c-suite [question]`

This is where I find it a little [chunibyo](https://en.wikipedia.org/wiki/Ch%C5%ABniby%C5%8D) when you say "let's call a C-level meeting!" Yet, I find it so useful to have a round-robin debate with several subagents. It actually refines your ideas from several perspectives (though each C-persona might need more time to develop). I call it when I want to get the big visionary direction of the project, that kind of thing. You can do something like:

```
/c-suite Should we position our v3.0 as a premium version or a free version with upgradeable features?
```

- Each agent (CEO, CTO, CMO, CFO, Product Lead) provides their perspective
- You respond after each agent—can add your verdict, rationale, or random thoughts there
- Say "synthesize" to get final recommendation
- Decision auto-saved to session logs

### `/session-log`

Intelligent session documentation that adapts to complexity and prevents recency bias. It serves 2 purposes: 1) as a journal to go back in time as the project grows, 2) to make the LLM credits worth it by logging them out so I don't need to ask them again later (I'm broke and short on budget lol).

```
/session-log
```

**What it does**:
- **Discovers artifacts**: Checks git history, plan files, prior sessions
- **Detects scope**: Automatically classifies as SMALL/MEDIUM/LARGE
- **Multi-pass for large sessions**: Creates outline first, then documents phases
- **Phase-aware**: Captures planning → implementation → debugging → refinement
- **Self-verifies**: Explicit checklist ensures nothing is missed
- **Execution log**: Tells complete story, not just recent work

**Session types**:
- SMALL (< 20 turns): Brief summary
- MEDIUM (20-50 turns): Standard comprehensive log
- LARGE (50+ turns / has plan / 5+ commits): Phase breakdown with full context

**Key improvement**: No longer suffers from recency bias - captures entire session including initial planning and early implementation, not just recent debugging.

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

This is a growing function from `session-log`. Normally with Claude plan mode, it will create a plan as markdown in `~/.claude/projects/` which is quite hard to find. This skill will create a plan as markdown in the project directory, which is more accessible, and more importantly, it can work across sessions. When Claude is dead (by context full or credit limit), you can still use the plan to continue your work later on easily. I think there might be a better way to use the existing plan mode to do this, and it might be more convenient in the future, yet I'm comfortable with this one lol.

```
/plan step-by-step adding authentication feature on v3.0
```

- Asks clarifying questions (purpose, rationale, timeline)
- Creates rationale-driven plan with task groups
- Exports to project directory

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

### Commands
Just another entry point to skills. Each skill above has a corresponding command (from the old days when skills didn't exist). It looks like Anthropic now prefers Skills, so slash commands might be deprecated later. I rarely trigger via commands now anyway.

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
