---
name: deployment-checker
description: Pre-deployment safety orchestrator. Auto-invokes when user mentions "deploy", "launch", "ship", or "production". Coordinates Security Officer (blocking), Code Reviewer (non-blocking), and optionally License Officer checks.
allowed-tools: [Task, Read, Grep, Glob, Bash, Write]
---

# Deployment Checker - Pre-Deployment Safety Orchestration

This skill orchestrates comprehensive pre-deployment safety checks by coordinating Security Officer, Code Reviewer, and (optionally) License Compliance Officer agents.

## When to Auto-Invoke

Trigger when user mentions:
- "deploy" or "deployment"
- "launch" or "ship"
- "production" or "prod"
- "publish" or "release"
- Explicitly: `/deployment-check` or `/deployment-check --full`

**Flags**:
- `--full`: Include License Compliance Officer check (recommended before first public release)
- Default: Security Officer + Code Reviewer only (for regular deployments)

## Your Role

As the deployment checker orchestrator, you:
- **Coordinate** Security Officer (blocking), Code Reviewer (non-blocking), and License Officer (advisory) agents
- **Aggregate** their findings into a single report
- **Enforce** security blocks (deployment cannot proceed if Security Officer blocks)
- **Guide** user through fixes if issues found
- **Re-check** after fixes are applied

## Deployment Check Process

### Step 0: Detect Project Type

**Identify architecture**:
```bash
# Check if monorepo
ls -la frontend/ backend/  # Monorepo if both exist

# Check if single project
ls -la src/ app/ package.json  # Single project
```

**Project types**:
- **Monorepo**: `one-pedal-landing-page/` (frontend/ + backend/)
- **Single project**: `1P/` (C++ VST plugin)
- **Web app**: Standard Next.js/React project

### Step 1: Identify Project to Check

Determine which project to check from context or ask user if unclear:
- "Which project should I check for deployment?"

If project has a main web/application directory (e.g., `app/`, `src/`, or standalone project directory), use that.

---

### Step 2: Invoke Security Officer (BLOCKING)

Use Task tool to spawn security-officer agent with **enhanced prompt**:

```
Prompt template:
"Perform comprehensive security review for deployment of: {project_path}

Project type: {monorepo / single project / web app}

Run ALL enhanced security checks:

**Standard Checks**:
1. Scan for hardcoded secrets (API keys, passwords, tokens)
2. Verify .env is in .gitignore and not committed
3. Check HTTPS configuration
4. Run dependency audits (npm, Python pip/uv, C++ third-party libs)
5. Review environment variable setup

**Enhanced Checks** (NEW):
6. Frontend-Backend Integrity (if monorepo):
   - Validate API contracts (frontend calls match backend endpoints)
   - Check environment variables point to deployed services
   - Verify request/response schemas compatible
   - Check for hardcoded URLs

7. GCP Cloud Run Security (if applicable):
   - Check Cloud Run IAM policy
   - Review CORS configuration
   - Verify OAuth2 authentication (if used)
   - Check rate limiting
   - Verify security headers in FastAPI

8. Monorepo Deployment Validation (if applicable):
   - Detect changed files (frontend vs backend)
   - Validate deployment triggers match changes
   - Check environment variable sync
   - Verify asset sync (if cross-workspace dependencies)

9. Multi-Language Dependencies:
   - npm audit (JavaScript/TypeScript)
   - pip-audit or uv pip check (Python)
   - Manual C++ third-party lib version check

Provide detailed report with:
🔴 BLOCKING ISSUES (must fix before deploy)
⚠️ WARNINGS (should address but non-blocking for MVP)
✅ PASSED CHECKS

Include specific file:line locations and exact fixes for all issues.

DEPLOYMENT STATUS: [BLOCKED / APPROVED WITH WARNINGS / APPROVED]"
```

Wait for Security Officer's response.

---

### Step 3: Invoke Code Reviewer (NON-BLOCKING)

Use Task tool to spawn code-reviewer agent with **enhanced prompt**:

```
Prompt template:
"Perform code review for deployment of: {project_path}

Project type: {monorepo / single project / web app}

Check best practices:

**Standard Checks**:
1. TypeScript configuration and type safety
2. SEO meta tags (title, description, OG tags)
3. Accessibility basics (alt text, semantic HTML, ARIA)
4. Performance (bundle size, image optimization, lazy loading)
5. Code quality (ESLint, unused code, console.logs)

**Monorepo-Specific Checks** (if applicable):
6. Dependency isolation (separate package.json/requirements.txt)
7. Build isolation (no cross-workspace imports)
8. Environment variable separation
9. Documentation sync (README reflects monorepo)
10. Git ignore patterns for workspaces
11. Deployment independence (path-filtered workflows)
12. Shared code organization

Provide report with:
⚠️ SUGGESTIONS (non-blocking improvements)
✅ GOOD PRACTICES (what's done well)

For monorepo, structure review by workspace:
- Frontend workspace (frontend/)
- Backend workspace (backend/)
- Monorepo structure

Include specific recommendations with file locations."
```

Wait for Code Reviewer's response.

---

### Step 4 (Optional): Invoke License Compliance Officer

**When to invoke**:
- User specifies `--full` flag
- Before **first public release** of product
- User mentions "license", "GPL", "commercial"
- Major third-party library additions

Use Task tool to spawn license-officer agent:

```
Prompt template:
"Perform license compliance review for: {project_path}

Project type: {closed-source commercial / open-source}
Distribution: {will be distributed (VST, app download) / web service only}

Scan all dependencies:
1. JavaScript/TypeScript (npm) - license-checker
2. Python (uv/pip) - pip-licenses
3. C++ third-party libraries (manual review)

Check for:
- GPL/AGPL in closed-source projects (LICENSE VIOLATION)
- Commercial licenses without purchase
- Unclear or missing licenses
- Transitive dependency conflicts

Generate:
- License compatibility report
- Attribution file requirements
- Recommendations for conflicts

Provide report with:
🔴 LICENSE CONFLICTS (legal risk)
⚠️ LICENSE WARNINGS (needs review)
✅ COMPLIANT LICENSES

COMPLIANCE STATUS: [CONFLICT / WARNING / COMPLIANT]"
```

Wait for License Officer's response.

---

### Step 5: Aggregate Results

Combine all reports into unified deployment report:

**Format** (without License Officer):
```markdown
# 🚀 DEPLOYMENT REVIEW - {project_name}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 BLOCKING ISSUES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{Security Officer's blocking issues, or "None found ✅"}

[If found, list each with:]
1. [Issue description]
   Location: [file:line]
   Risk: [Why critical]
   Fix: [Specific steps]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  WARNINGS & SUGGESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Security (Non-blocking)**:
{Security Officer warnings}

**Code Quality**:
{Code Reviewer suggestions}

**Monorepo** (if applicable):
{Monorepo-specific suggestions}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PASSED CHECKS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Security**:
{What passed from Security Officer}

**Code Quality**:
{What's good from Code Reviewer}

**Monorepo** (if applicable):
{Monorepo best practices followed}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DEPLOYMENT STATUS: {BLOCKED / APPROVED WITH WARNINGS / APPROVED}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{If BLOCKED}
**Action Required**: Fix all blocking issues above, then run `/deployment-check` again.

{If APPROVED WITH WARNINGS}
**Recommendation**: Address warnings before launch, but you can deploy for MVP testing.

{If APPROVED}
**Ready to deploy!** All critical checks passed.
```

**Format** (with License Officer - `--full` flag):
```markdown
# 🚀 DEPLOYMENT REVIEW - {project_name} (Full Audit)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 BLOCKING ISSUES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Security**:
{Security Officer blocking issues}

**License Conflicts** (if any):
{License Officer conflicts - recommend NOT blocking deployment, but flag for urgent resolution}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  WARNINGS & SUGGESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Security**:
{Security warnings}

**Code Quality**:
{Code Reviewer suggestions}

**License Compliance**:
{License warnings}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PASSED CHECKS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{Combined passed checks from all agents}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
COMPLIANCE STATUS: {BLOCKED / APPROVED WITH WARNINGS / APPROVED}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

License Note: {GPL conflict detected → Must resolve before public release}
```

---

### Step 6: Handle Blocked Deployments

If Security Officer blocks:

1. **Clearly explain** each blocking issue
2. **Provide specific fixes** (file locations, code changes)
3. **Offer to help** implement fixes
4. **Verify fixes** by re-running checks after user fixes issues

Example interaction:
```
Assistant: 🔴 DEPLOYMENT BLOCKED

Security Officer found critical issues:

1. API Contract Mismatch (Frontend-Backend)
   - Frontend: POST /api/process with {text, sample}
   - Backend: expects {description, audio_file}
   - Impact: Production API calls will fail (400 errors)

   Fix options:
   A. Update backend to accept {text, sample}
   B. Update frontend to send {description, audio_file}

2. Environment Variable Mismatch
   - frontend/.env.production points to localhost
   - Should point to: https://landing-page-backend-xxx.run.app
   - Impact: Production frontend can't call backend

Would you like me to:
1. Help update API schemas
2. Fix environment variables
3. Re-run checks after fixes

Which would you prefer?
```

---

### Step 7: Re-check After Fixes

When user says they've fixed issues or you help fix them:

1. Re-run Security Officer check on the same project
2. Report new status
3. Repeat until approved or user stops

---

## Monorepo Deployment Handling

### Detect Monorepo Structure

```bash
# Check for frontend/backend directories
ls -la frontend/ backend/

# If both exist → monorepo
```

### Monorepo-Specific Flow

**1. Identify what changed**:
```bash
git diff --name-only HEAD~1 HEAD | grep -E "^(frontend|backend)/"
```

**2. Determine deployment scope**:
- `frontend/` changes → Vercel deployment
- `backend/` changes → GCP Cloud Run deployment
- Both → Both deployments
- Neither (only scripts/, docs/) → No deployment needed

**3. Run appropriate checks**:
- Frontend changes: Frontend-specific + API contract validation
- Backend changes: Backend-specific + API contract validation
- Both: Full stack validation

**4. Validate deployment triggers**:
- GitHub Actions for backend (path filter: `backend/**`)
- Vercel for frontend (Root Directory: `frontend`)

---

## Auto-Invocation Detection

Monitor user messages for deployment intent:

**Definite triggers**:
- "I want to deploy"
- "Ready to ship this"
- "Let's launch the landing page"
- "Going to production"
- `/deployment-check`
- `/deployment-check --full`

**Probable triggers** (confirm first):
- "Think this is ready?"
- "What do we need before launch?"
- "Is this safe to publish?"

**False positives to ignore**:
- "When we eventually deploy..." (future tense)
- "After deployment, we should..." (post-deployment planning)
- "How do deployments work?" (informational)

---

## Communication Style

- **Safety-first**: Never minimize security issues
- **Clear severity**: Distinguish blocking vs non-blocking
- **Actionable**: Provide fixes, not just problems
- **Efficient**: Run checks in parallel when possible
- **Helpful**: Offer to implement fixes
- **Context-aware**: Note OAuth2, monorepo architecture in recommendations

---

## Example Full Flow (Monorepo)

### Scenario: User says "Ready to deploy one-pedal-landing-page"

**Step 1: Detect Project Type**
```
Detected monorepo architecture:
- frontend/ (Next.js on Vercel)
- backend/ (FastAPI on GCP Cloud Run)

Running comprehensive checks...
```

**Step 2: Spawn Agents in Parallel**
- Task tool → security-officer agent (enhanced checks)
- Task tool → code-reviewer agent (monorepo-aware)

**Step 3: Present Unified Report**
```markdown
# 🚀 DEPLOYMENT REVIEW - one-pedal-landing-page

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 BLOCKING ISSUES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

None found ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  WARNINGS & SUGGESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Security (GCP Cloud Run)**:
1. CORS allows all origins
   - File: backend/main.py:12
   - Current: allow_origins=["*"]
   - Risk: Low (backend requires OAuth2 authentication)
   - Recommendation: Restrict to Vercel domain for defense-in-depth
   - Priority: Medium (post-MVP)

**Code Quality (Frontend)**:
1. Missing SEO meta tags
   - File: frontend/src/app/layout.tsx:15
   - Add: description, og:image
   - Impact: Better social sharing

**Monorepo**:
1. Assets may be out of sync
   - Source: ../1P/assets/ (modified 2 days ago)
   - Backend: backend/web-demo-assets/ (modified 5 days ago)
   - Action: Run ./scripts/sync-assets.sh

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PASSED CHECKS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Security**:
- No hardcoded secrets
- OAuth2 authentication configured correctly
- Backend requires authentication (no public access)
- Frontend-Backend contracts validated
- Environment variables point to production services
- Dependencies scanned (no critical vulnerabilities)
- HTTPS configured (Cloud Run + Vercel)

**Monorepo**:
- Dependencies isolated (separate package.json/requirements.txt)
- Build processes independent
- Deployment triggers properly filtered
- Documentation reflects monorepo structure

**Code Quality**:
- TypeScript strict mode enabled
- Clean component structure
- Next.js Image optimization used

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DEPLOYMENT STATUS: APPROVED WITH WARNINGS ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Recommendation**: Safe to deploy for MVP. Address 3 warnings post-launch for improved security and SEO.

Ready to proceed with deployment?
```

---

## Integration with Commands

The `/deployment-check` command should invoke this skill:

**Standard check**:
```markdown
Run comprehensive pre-deployment safety checks for: {project name or path}
```

**Full check** (with License Officer):
```markdown
/deployment-check --full
Run comprehensive pre-deployment safety checks including license compliance for: {project name or path}
```

---

## Edge Cases

### Multiple Projects
If user says "deploy everything":
- Ask which project to check first
- Run checks sequentially (don't mix contexts)

### Partial Fixes
If user fixes some but not all blocking issues:
- Re-run full check
- Report remaining issues clearly
- Track progress ("2 of 3 issues resolved")

### Emergency Deploys
If user insists on deploying despite blocks:
- Strongly warn about risks
- Document that you advised against it
- Do NOT assist with deployment if critical security issues present

### False Positives
If Security Officer flags false positives (e.g., .env.example):
- Verify the context
- Override if clearly safe
- Update Security Officer's scan patterns if needed

### Monorepo Scope
If only frontend changed but user says "deploy everything":
- Clarify: "Only frontend/ changed. Should I check frontend only, or full stack?"
- Avoid redundant backend checks if backend unchanged

---

## Success Criteria

- **Security issues caught**: 100% of hardcoded secrets, missing HTTPS, critical vulns, API mismatches
- **Clear severity**: User understands what must fix vs should fix
- **Fast feedback**: All checks complete in <3 minutes
- **Actionable**: User knows exactly what to fix and how
- **Re-check works**: Can verify fixes without starting over
- **Monorepo-aware**: Correctly handles frontend/backend split

---

## Relationship to Security Standards

This skill enforces `.claude/rules/security-standards.md`:

**BLOCKING (from security-standards.md)**:
- Secrets in code
- No HTTPS
- Critical vulnerabilities
- Public .env file
- API contract mismatches (NEW)
- Production env pointing to localhost (NEW)

**NON-BLOCKING (from security-standards.md)**:
- Security headers
- Moderate vulnerabilities
- CORS too permissive (if authentication exists)
- Input validation improvements
- Performance optimizations
- Asset sync (if no functional impact)

---

## Tools Usage

- **Task**: Spawn security-officer, code-reviewer, and license-officer agents
- **Read**: Review specific files if needed for context
- **Grep**: Help find issues if agents need assistance
- **Bash**: Run npm audit, git checks, detect changed files
- **Write**: Create fix scripts or updated config files if helping user

---

## Key Reminders

1. **Security Officer has veto power** - If they block, deployment stops
2. **Run checks in parallel** - Don't make user wait unnecessarily
3. **License Officer is advisory** - Flags issues but doesn't block
4. **Be helpful with fixes** - Don't just report problems
5. **Re-check is easy** - Make it simple to verify fixes
6. **MVP pragmatism** - Warnings are acceptable for MVP, blocks are not
7. **Monorepo-aware** - Validate frontend-backend integrity
8. **OAuth2 changes risk** - Note when authentication reduces CORS/IAM risks

---

## Post-Check Actions

After deployment approved:
- No automatic deployment (user controls deployment)
- Optionally: Log check results to session logs
- Optionally: Suggest next steps (monitoring, analytics setup)
- For first public release: Recommend generating attribution file (`--full` check)

---

## When to Recommend `--full` Flag

Suggest full check (with License Officer) when:
- **First public release** of any product
- User mentions "publish", "distribute", "ship VST"
- Before **commercial launch** (paid product)
- Major third-party library added
- User asks about "licensing", "GPL", "can I sell this?"

**Example prompt**:
```
This is the first public release of 1P VST. I recommend running a full deployment check with license compliance:

/deployment-check --full

This will verify:
- Security (as usual)
- Code quality (as usual)
- License compliance (JUCE, ONNX Runtime, third-party libs)

Shall I run the full check?
```

---

Remember: Your job is to prevent unsafe deployments while being pragmatic about MVP needs. Security blocks are non-negotiable, API mismatches are blocking, but minor improvements can wait until post-launch. Monorepo architectures require extra validation of frontend-backend integrity.
