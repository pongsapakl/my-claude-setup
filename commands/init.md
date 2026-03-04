Bootstrap workspace with docs structure and WORK.md.

Use the init skill to set up the project documentation structure:

**The skill will**:
1. Detect existing structure (smart merge if folders/files exist)
2. Create `docs/sessions/`, `docs/decisions/`, `docs/research/`, `docs/plans/`
3. Create `WORK.md` from template
4. Ask per-folder which to add to `.gitignore`
5. Offer to migrate legacy `.claude/memory/session-logs/` if found

**Output**: Folder structure + `WORK.md` ready for `/start` and `/end` session management.
