# My Claude Team

[Claude when you update readme please dont edit this part. This is handwritten (or handtyped) and I wanted to keep it as is. Below ## Quick Install you can do whatever you are born to do.]

It appears to me that there are so many productivity frameworks out there trying to utilize the function of agents, skills, etc. as provided by Claude. Yet I find many of them are more generic and load everything up front (which I find somewhat annoying) and not tailored to my own needs. So, I created a custom one based on those frameworks out there and made it fit my requirements a little more. It is very basic yet so powerful for my workflow.

This repo serves as my archive/backup for my workflow (integrated with Claude Code, obviously), yet I try to make it more generic by moving project-specific information to the `CLAUDE.md` file and letting agents read from there to make this tool more reusable for other projects. If you find this interesting, please feel free to test it out, fork it—comments are appreciated, yet I can't confirm I'll fix issues since it is my workflow for my needs after all. Also feel free to basically fetch this to Claude and let it generate your own version of this. Well, maybe fetching some other repo might provide more polished ideas, but whatever.

[UPDATE v.0.5.0] I also feels like handing over session for me is really important. Despite there are things like `/compact` it is still not as good as i think it can be. That is why I use a file-based system to hand over session. This way, not only agent, but also us humans can catch up on what is left, what to do next more easily. Key is what is just done, and what to do next (along with related file).

## Quick Install

```bash
/plugin marketplace add pongsapakl/my-claude-team
/plugin install my-claude-team
```

## What Problem Does This Solve?

Claude Code sessions are ephemeral. When a session ends — whether from context limits, credit caps, or just closing the terminal — all the context about what you were doing, what decisions you made, and what comes next is gone. The next session starts from zero.

This plugin fixes that with a **file-based handover system**. At the end of every session, `/end` writes down what was done, what's half-finished, and — most importantly — a specific, actionable note for the next session to pick up from. At the start of the next session, `/start` reads that note and gets you (or Claude) back up to speed in seconds.

It also gives you tools to make better decisions (`/c-suite`, `/research`) and keep a project's history organized (`docs/decisions/`, `docs/research/`, `docs/plans/`).

## How It Works

```text
  ┌─────────────────────────────────────────────────────────────┐
  │                     SESSION LIFECYCLE                        │
  │                                                             │
  │   /init (once)         /start            /end               │
  │   ┌──────────┐      ┌──────────┐      ┌──────────┐         │
  │   │ Set up   │      │ Read     │      │ Write    │         │
  │   │ docs/    │      │ WORK.md  │      │ session  │         │
  │   │ folders  │      │ + latest │      │ log +    │         │
  │   │ + WORK.md│      │ session  │      │ update   │         │
  │   └──────────┘      │ log      │      │ WORK.md  │         │
  │        │            └────┬─────┘      └────┬─────┘         │
  │        │                 │                  │               │
  │        ▼                 ▼                  ▲               │
  │   ┌──────────────────────────────────────────────┐         │
  │   │                YOUR WORK                      │         │
  │   │                                               │         │
  │   │  /research ──► docs/research/                 │         │
  │   │  /c-suite  ──► docs/decisions/                │         │
  │   │  /plan     ──► docs/plans/                    │         │
  │   │  coding, debugging, etc.                      │         │
  │   └───────────────────────────────────────────────┘         │
  └─────────────────────────────────────────────────────────────┘

  SESSION 1                          SESSION 2
  ┌────────────────────┐             ┌────────────────────┐
  │ /start             │             │ /start             │
  │   ↓                │             │   ↓                │
  │ (work)             │  WORK.md    │ (work)             │
  │   ↓                │ ─────────►  │   ↓                │
  │ /end               │  + session  │ /end               │
  │  "Next: Wire up    │    log      │  "Next: ..."       │
  │   the webhook..."  │             │                    │
  └────────────────────┘             └────────────────────┘
```

## Typical Workflow

Here's what using this looks like day-to-day:

**First time on a project** — run `/init` once:

```text
> /init
✓ Created docs/sessions/, docs/decisions/, docs/research/, docs/plans/
✓ Created WORK.md
✓ Added docs/sessions/ to .gitignore
```

**Starting a session** — run `/start`:

```text
> /start

Phase: Building auth system
Last session (2026-03-03): Implemented JWT middleware, wrote login endpoint
Resuming from: "Add refresh token rotation to auth/tokens.ts — the access token
  flow works but refresh is stubbed out. Schema is in docs/decisions/2026-03-02-jwt-strategy.md"
Open questions: none

What would you like to focus on?
```

**During a session** — work normally, use other skills as needed:

```text
> /research should we use Redis or in-memory for token blacklist?
> /c-suite should we add social login for v1 or keep email-only?
> /plan add password reset flow
```

**Ending a session** — run `/end`:

```text
> /end

Done:
- Implemented refresh token rotation
- Added token blacklist (chose Redis — see docs/decisions/2026-03-04-redis-blacklist.md)

Half-done:
- Password reset flow — email sending works, reset page not built yet

Next session starts with:
"Build the reset password page at app/auth/reset/page.tsx — the API endpoint
  POST /api/auth/reset exists and sends emails. Need the form UI that calls it
  and handles the token from the URL query param."

✓ Session logged: docs/sessions/2026-03-04-auth-tokens.md
✓ WORK.md updated
```

**Next session** (even days later, even a different model) — run `/start` and it picks up exactly there.

## Getting Started

After installing, run `/init` in your project to set up the workspace:

```text
your-project/
├── TODO.md                  ← your scratchpad (human-facing, append-only)
├── WORK.md                  ← AI context (multi-track state, merge-updated)
├── docs/
│   ├── sessions/            ← rich session logs (written by /end)
│   ├── decisions/           ← ADRs (written by any skill, anytime)
│   ├── research/            ← research spikes (written by /research)
│   └── plans/               ← implementation plans (written by /plan)
└── CLAUDE.md                ← your project context (you write this)
```

## Skills Reference (8 total)

### Session Management — `/init`, `/start`, `/end`

These three skills form the core workflow. They keep context across sessions so you (and Claude) never lose track of where things are.

| Skill | When | What it does |
|-------|------|-------------|
| `/init` | Once per project | Creates `docs/` folders + `WORK.md` + `TODO.md`, asks about tracks and gitignore |
| `/start` | Beginning of session | Shows track overview, reads `TODO.md` highlights + latest session log, asks which track to focus on |
| `/end` | End of session | Writes rich narrative session log, merge-updates only the active track in `WORK.md`, appends to `TODO.md` |

**Key innovations in v0.6.0:**
- **Multi-track WORK.md** — parallel workstreams (frontend, backend, etc.) don't overwrite each other. `/end` only updates the track you worked on.
- **TODO.md scratchpad** — your freeform, append-only human-readable task tracker. Claude appends, never deletes.
- **Rich narrative session logs** — detailed "What Happened" stories, not just checkboxes. Includes alternatives explored, technical details, and lessons learned.
- **Default-write artifacts** — decisions and research docs are drafted by default at session end (opt-out, not opt-in).
- **Mid-session documentation** — prompts to capture decisions and research immediately when they happen, not just at `/end`.

### Strategic — `/c-suite`, `/research`, `/plan`

| Skill | Example | What it does |
|-------|---------|-------------|
| `/c-suite` | `/c-suite Should we go freemium or paid?` | Round-robin debate with CEO, CTO, CMO, CFO, Product Lead agents. You respond after each. Decisions saved to `docs/decisions/` |
| `/research` | `/research can we do real-time audio in browser?` | Auto-detects quick vs deep scope. Deep mode invokes relevant C-level agents. Findings saved to `docs/research/` |
| `/plan` | `/plan add authentication to v3.0` | Creates implementation plan with rationale. Saved to `docs/plans/`, referenced from `WORK.md` |

### Ops — `/deployment-check`, `/infra-check`

| Skill | What it does |
|-------|-------------|
| `/deployment-check` | Pre-deploy safety: Security Officer (blocking) + Code Reviewer (warnings). Use `--full` for license compliance |
| `/infra-check` | Config drift detection: compares actual GCP/Vercel settings vs CLAUDE.md docs |

## Agents

8 agents power the skills above. They can also be activated individually if needed.

| Agent | Focus | Impact |
|-------|-------|--------|
| **CEO** | Strategy, market fit | Advisory |
| **CTO** | Technical architecture, scalability | Advisory |
| **CMO** | Marketing, messaging, acquisition | Advisory |
| **CFO** | Financial, ROI, pricing | Advisory |
| **Product Lead** | UX, features, prioritization | Advisory |
| **Security Officer** | Deployment safety, vulnerabilities | **Blocking** |
| **Code Reviewer** | Code quality, performance, SEO | Warnings |
| **License Officer** | GPL compliance, attribution | Advisory |

## Rules

Plugin-level rules are loaded automatically when installed:

- **docs-structure** — three-file architecture (TODO.md / WORK.md / session logs), when to write ADRs, mid-session documentation
- **session-workflow** — multi-track `/start` → `/end` lifecycle, track-scoped updates, handoff quality criteria
- **cli-workflow** — use CLI tools over MCP servers to save tokens
- **discussion-protocol** — how C-Suite round-robin debates work
- **git-commit-workflow** — commit conventions
- **security-standards** — non-negotiable security checks

## Customization

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
