# CLAUDE.md

## Project Overview

**my-claude-team** is a Claude Code plugin that provides a multi-track session handover system and virtual C-suite advisory team. It solves the problem of ephemeral Claude Code sessions by persisting context across sessions via a three-file architecture: TODO.md (human scratchpad), WORK.md (AI context), and rich narrative session logs.

## Project Structure

```
my-claude-team/
├── agents/          # 8 agent definitions (CEO, CTO, CMO, CFO, Product Lead, Security Officer, Code Reviewer, License Officer)
├── skills/          # 8 skills: init, start, end, plan, research, c-suite-meeting, deployment-checker, infra-checker
├── rules/           # 6 plugin rules: docs-structure, session-workflow, cli-workflow, discussion-protocol, git-commit-workflow, security-standards
├── .claude-plugin/  # Plugin manifest (plugin.json)
├── LICENSE
└── README.md
```

## Key Concepts

- **Three-file architecture**: TODO.md (human), WORK.md (AI), session logs (both)
- **Multi-track WORK.md**: Parallel workstreams don't clobber each other; /end only updates the active track
- **TODO.md**: Append-only freeform scratchpad; Claude never deletes user content
- **Rich session logs**: Narrative "What Happened" stories, not just checkboxes
- **Session lifecycle**: /init (once) -> /start (begin, pick track) -> work -> /end (close, merge-update track)
- **Mid-session documentation**: Decisions and research captured immediately, not just at /end
- **Default-write artifacts**: ADRs and research docs drafted by default (opt-out, not opt-in)

## Development Notes

- Plugin is installed via: `/plugin marketplace add pongsapakl/my-claude-team`
- Skills are defined in `skills/<name>/SKILL.md` files
- Agents are markdown files in `agents/`
- Rules are markdown files in `rules/`
- Version is tracked in `.claude-plugin/plugin.json`
- The README has a hand-written intro section that should not be edited by Claude (marked with a comment)
