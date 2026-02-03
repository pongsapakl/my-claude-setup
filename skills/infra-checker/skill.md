---
name: infra-checker
description: Infrastructure Integrity Checker - validates cloud infrastructure configuration matches documentation. Auto-invokes when user mentions "infrastructure", "config drift", or "/infra-check".
user-invocable: true
auto-invoke-on:
  - infrastructure
  - infra
  - config drift
  - "/infra-check"
  - infrastructure audit
---

# Infrastructure Integrity Checker Skill

## What This Skill Does

Validates that actual cloud infrastructure configuration (GCP, Vercel, GitHub Actions) matches documented configuration in CLAUDE.md files.

**Purpose**: Detect **configuration drift** - when infrastructure changes but documentation doesn't (or vice versa).

## When to Use This Skill

- **Quarterly audits** (every 3 months)
- Before **major architecture changes**
- After **infrastructure incidents** (outages, misconfigurations)
- User mentions "infrastructure", "config drift", "/infra-check"
- Onboarding **new team members** (verify docs accurate)

## How It Works

### Step 0: Understand Request

**Identify what to check**:
- Specific project? (e.g., "check one-pedal-landing-page infrastructure")
- All projects? (e.g., "audit all infrastructure")
- Specific service? (e.g., "check GCP Cloud Run config")

**Default**: Check all projects in workspace if not specified.

---

### Step 1: Find CLAUDE.md Documentation

```bash
# Find all CLAUDE.md files
find /Users/pongsapakl/Documents/Projects/GenDSP -name "CLAUDE.md" -type f
```

**Expected locations**:
- `GenDSP/.claude/CLAUDE.md` (workspace-level)
- `one-pedal-landing-page/.claude/CLAUDE.md` (project-specific)
- `1P/.claude/CLAUDE.md` (project-specific)

**Read each** and extract documented infrastructure:
- GCP project IDs, regions, service names
- Vercel project names, environment variables
- GitHub Actions workflows
- Service account emails, IAM roles

---

### Step 2: Query Actual Infrastructure (GCP)

For each GCP project documented in CLAUDE.md:

#### 2.1: Cloud Run Services

```bash
# List all Cloud Run services
gcloud run services list --region=asia-northeast1 --format=json

# For each service, get details
gcloud run services describe <service-name> \
  --region=<region> \
  --format=json
```

**Extract**:
- Service name
- Region
- Memory (e.g., `512Mi`)
- CPU (e.g., `1`)
- Max/Min instances
- Environment variables (names only, not values)
- Service account
- IAM policy (who can invoke)

#### 2.2: Artifact Registry

```bash
# List repositories
gcloud artifacts repositories list --location=<region> --format=json
```

**Extract**:
- Repository name
- Location/region
- Format (Docker, etc.)

#### 2.3: Workload Identity

```bash
# List Workload Identity pools
gcloud iam workload-identity-pools list --location=global --format=json

# For each pool, list providers
gcloud iam workload-identity-pools providers list \
  --workload-identity-pool=<pool-name> \
  --location=global \
  --format=json
```

**Extract**:
- Pool name
- Provider name
- Attribute mappings (GitHub repo bindings)

#### 2.4: Service Accounts & IAM

```bash
# List service accounts
gcloud iam service-accounts list --format=json

# For each service account, get IAM policy
gcloud projects get-iam-policy <project-id> \
  --flatten="bindings[].members" \
  --format="table(bindings.role, bindings.members)" \
  --filter="bindings.members:<service-account-email>"
```

**Extract**:
- Service account emails
- Roles assigned

---

### Step 3: Query Actual Infrastructure (Vercel)

```bash
# Get Vercel project details
vercel project ls --format json

# For specific project
vercel env ls <project-name> --format json
```

**Extract**:
- Project name
- Production domain
- Environment variables (names only, not values)
- Build settings (framework, root directory)

**Note**: Vercel CLI may require authentication. If not available, mark as "Manual Verification Needed".

---

### Step 4: Query GitHub Actions Workflows

```bash
cd <project-directory>

# List workflow files
ls -la .github/workflows/*.yml

# For each workflow, extract:
cat .github/workflows/<workflow>.yml
```

**Extract**:
- Workflow name
- Trigger conditions (branches, paths)
- Secrets used
- GCP project/service deployed to
- Docker registry URLs

---

### Step 5: Compare Documented vs Actual

**For each documented item**, check if it matches actual:

| Item | Documented (CLAUDE.md) | Actual (API Query) | Status |
|------|----------------------|-------------------|--------|
| **GCP Project ID** | one-pedal-landing-page | one-pedal-landing-page | ✅ MATCH |
| **Cloud Run Region** | asia-northeast1 | asia-northeast1 | ✅ MATCH |
| **Cloud Run Memory** | 512Mi | 1Gi | ⚠️ DRIFT |
| **Service Account** | github-actions@... | github-actions@... | ✅ MATCH |
| **Vercel Project** | one-pedal-landing-page | one-pedal-landing-page | ✅ MATCH |
| **Vercel Domain** | one-pedal-landing-page.vercel.app | one-pedal-landing-page.vercel.app | ✅ MATCH |

**Drift Detected When**:
- Documented value ≠ Actual value
- Documented item doesn't exist in actual infrastructure
- Actual item not documented

---

### Step 6: Generate Drift Report

**Report Format**:

```
🔍 INFRASTRUCTURE INTEGRITY CHECK

Project: one-pedal-landing-page
Audit Date: 2026-02-02
Auditor: Infrastructure Integrity Checker

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  CONFIGURATION DRIFT DETECTED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Cloud Run Service: landing-page-backend
   Component: Memory allocation
   Documented: 512Mi (CLAUDE.md line 54)
   Actual: 1Gi
   Drift: Memory increased (unknown when/why)

   Actions:
   A. Update documentation: one-pedal-landing-page/.claude/CLAUDE.md line 54
   B. Or revert config: gcloud run services update landing-page-backend --memory=512Mi --region=asia-northeast1

   Recommendation: Update documentation (infrastructure likely intentionally scaled)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ CONFIGURATION MATCHES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GCP Cloud Run:
- ✅ Service name: landing-page-backend
- ✅ Region: asia-northeast1
- ✅ CPU: 1
- ✅ Max Instances: 10
- ✅ Min Instances: 0

Artifact Registry:
- ✅ Repository: landing-page-demo
- ✅ Location: asia-northeast1
- ✅ Format: Docker

Workload Identity:
- ✅ Pool: github-actions
- ✅ Provider: github
- ✅ Service Account: github-actions@one-pedal-landing-page.iam.gserviceaccount.com

Vercel:
- ✅ Project: one-pedal-landing-page
- ✅ Domain: one-pedal-landing-page.vercel.app

GitHub Actions:
- ✅ Workflow: deploy-backend.yml
- ✅ Paths filter: backend/**

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DRIFT STATUS: ⚠️  MINOR DRIFT (1 item)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Severity: LOW (documentation out of date, no functional impact)
Action Required: Update CLAUDE.md to reflect actual memory allocation

Next Audit: 2026-05-02 (quarterly)
```

---

### Step 7: Recommend Actions

**For each drift item**:
1. **Assess severity**:
   - 🔴 **CRITICAL**: Security misconfiguration (public IAM, wrong region)
   - ⚠️ **MEDIUM**: Performance impact (memory, CPU, instances)
   - 📝 **LOW**: Documentation only (no functional impact)

2. **Provide options**:
   - **Option A**: Update documentation to match actual
   - **Option B**: Revert infrastructure to match documentation
   - **Option C**: Accept drift (if intentional)

3. **Recommend**:
   - Usually: Update documentation (infrastructure likely changed for good reason)
   - Exception: If actual config is wrong, revert infrastructure

---

## Example Drift Scenarios

### Scenario 1: Memory Increased Without Documentation

**Drift**:
- Documented: `512Mi`
- Actual: `1Gi`

**Likely Cause**: Performance tuning after deployment

**Recommendation**: Update documentation (infrastructure change likely intentional)

**Action**:
```markdown
# one-pedal-landing-page/.claude/CLAUDE.md line 54

# Before:
- Memory: 512Mi

# After:
- Memory: 1Gi (increased 2026-01-15 for better performance)
```

---

### Scenario 2: Wrong Region (Critical)

**Drift**:
- Documented: `asia-northeast1` (Tokyo)
- Actual: `us-central1` (Iowa)

**Likely Cause**: Deployment error or documentation error

**Severity**: 🔴 **CRITICAL** (wrong region = higher latency, wrong costs)

**Recommendation**: Verify intended region with user, then either:
- Fix documentation if `us-central1` is correct
- Migrate service to `asia-northeast1` if Tokyo is required

**Action**:
```bash
# If Asia required, migrate service
gcloud run services update landing-page-backend \
  --region=asia-northeast1
```

---

### Scenario 3: Undocumented Environment Variable

**Drift**:
- Documented env vars: `BACKEND_URL`, `GCP_SERVICE_ACCOUNT_KEY`
- Actual env vars: `BACKEND_URL`, `GCP_SERVICE_ACCOUNT_KEY`, `DEBUG_MODE`

**Likely Cause**: Debugging added, forgot to document

**Severity**: 📝 **LOW** (unless `DEBUG_MODE=true` in production 🔴)

**Recommendation**:
1. Check if `DEBUG_MODE` should be in production
2. Document in CLAUDE.md or remove from production

---

### Scenario 4: Service Account Role Drift

**Drift**:
- Documented roles: `roles/run.admin`, `roles/artifactregistry.writer`
- Actual roles: `roles/run.admin`, `roles/artifactregistry.writer`, `roles/editor`

**Severity**: 🔴 **CRITICAL** (over-permissioned)

**Recommendation**: Remove `roles/editor` (too broad, violates least privilege)

**Action**:
```bash
gcloud projects remove-iam-policy-binding one-pedal-landing-page \
  --member='serviceAccount:github-actions@one-pedal-landing-page.iam.gserviceaccount.com' \
  --role='roles/editor'
```

---

## What to Skip (Out of Scope)

- **Secrets/Credentials**: Don't validate actual values (security risk)
- **Code**: This skill checks infrastructure, not code
- **Dependencies**: Use Security Officer for dependency checks
- **Licenses**: Use License Compliance Officer

**This skill focuses on**: Infrastructure config (regions, memory, IAM roles, service names)

---

## Tools Needed

- **gcloud CLI**: Query GCP infrastructure
- **vercel CLI**: Query Vercel projects (if available)
- **git**: Access GitHub Actions workflows
- **Read**: Read CLAUDE.md files
- **Grep**: Extract config from CLAUDE.md
- **Bash**: Run queries and comparisons

---

## Permissions Required

```json
{
  "permissions": {
    "allow": [
      "Bash(gcloud:*)",
      "Bash(vercel:*)",
      "Bash(cat:*)",
      "Bash(ls:*)",
      "Read",
      "Grep"
    ]
  }
}
```

---

## Communication Style

- **Clear & Factual**: Report drift without alarm
- **Context-Aware**: Distinguish critical vs minor drift
- **Actionable**: Provide exact commands to fix
- **Educational**: Explain why drift matters
- **Pragmatic**: Usually recommend updating docs (infrastructure changes for good reasons)

---

## Frequency Recommendations

- **Quarterly**: Scheduled audits (every 3 months)
- **After incidents**: Infrastructure changes, outages
- **Before major changes**: Validate baseline before modifications
- **Ad-hoc**: User request (`/infra-check`)

---

## Output Files

### Create Audit Log

After each check, save audit log:

```markdown
# .claude/memory/infra-audits/2026-02-02-infrastructure-audit.md

# Infrastructure Audit - 2026-02-02

## Projects Audited
- one-pedal-landing-page
- 1P VST (no cloud infrastructure)

## Drift Summary
- 1 minor drift detected (memory allocation)
- 0 critical issues

## Actions Taken
- Updated CLAUDE.md to reflect actual memory (1Gi)

## Next Audit
- Scheduled: 2026-05-02
```

---

## When to Escalate to User

- **Critical drift**: Wrong region, over-permissioned roles
- **Unknown drift**: Can't determine if intentional
- **Undocumented infrastructure**: Found resources not in CLAUDE.md
- **Security risks**: Public IAM, missing authentication

---

## Success Metrics

- **Drift detection rate**: % of audits finding drift
- **Documentation accuracy**: % of config matching
- **Audit frequency**: Quarterly audits completed on schedule
- **Critical drift resolution**: Time to fix critical issues

---

## Example Audit Flow

**User**: `/infra-check one-pedal-landing-page`

**Skill executes**:
1. ✅ Read `one-pedal-landing-page/.claude/CLAUDE.md`
2. ✅ Query GCP: `gcloud run services describe landing-page-backend`
3. ✅ Query Vercel: `vercel project ls`
4. ✅ Read `.github/workflows/deploy-backend.yml`
5. ✅ Compare documented vs actual
6. ✅ Generate drift report
7. ✅ Save audit log to `.claude/memory/infra-audits/`
8. ✅ Present recommendations to user

**Output**: Drift report with actions (update docs or revert config)

---

## Integration with Other Skills

- **Deployment Checker**: Optionally run infra-check before deployments
- **Security Officer**: Escalate IAM drift as security issue
- **Session Log**: Log audit results in session logs

---

## Limitations

- **No secrets validation**: Don't check actual env var values (security)
- **Manual Vercel check**: If Vercel CLI unavailable, prompt user to verify manually
- **No cost validation**: Doesn't check if config is cost-optimal (out of scope)
- **Documentation-driven**: Assumes CLAUDE.md is source of truth

---

Remember: Configuration drift is normal in evolving infrastructure. The goal is **visibility**, not enforcement. Help user maintain accurate documentation so future sessions don't re-discover the same infrastructure details.
