Create an implementation plan for: {feature/project, e.g., "voice input to [project-name]" or "user authentication"}

Use the plan skill to generate a structured implementation plan:

**The skill will**:
1. Gather context (session logs, existing code, related plans)
2. Ask clarifying questions:
   - What problem does this solve? (purpose/value)
   - Why this approach vs alternatives? (rationale)
   - What's the success metric?
   - Any constraints or non-goals?
   - Timeline?
3. Structure plan with task groups, files, dependencies, rationale
4. Export to `docs/plans/` and update WORK.md Active Plan field

**Output**:
- Plan file: `docs/plans/YYYY-MM-DD-topic.md` (single source of truth)
- WORK.md updated with Active Plan path
- Links to related session logs and architectural decisions

**Plan includes**:
- Task groups with "Goal" and "Rationale" (explains WHY)
- File-level specificity (exact paths to create/modify)
- Success criteria and dependencies
- Verification checklist
- "For Next Session" notes (cross-session continuity)

**To update existing plan**: Just say "update the plan" or "update the {project} plan" (no flags needed).

Ready to start implementation after reviewing the plan.
