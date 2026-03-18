---
description: Merge approved PR, verify deployment including security checks, sync branches, finalize compliance
---

# Deploy to Main

**Pipeline Stage:** 5 of 5
**Previous:** `4-submit-for-review.md` (after PR approved and CI passed)
**References:** Test Plan (post-deploy verification, DR targets)

---

## Prerequisites

- PR approved by reviewer
- All CI checks passed (enforced by branch protection)
- No unresolved review comments

## Steps

### Step 1: Merge the PR

**Option A: GitHub CLI (Preferred)**
```bash
gh pr list --head develop --json number --jq '.[0].number'
gh pr merge [PR-NUMBER] --merge --delete-branch=false
```

**Option B: GitHub Web UI**
1. Open PR → **Merge pull request** → "Create a merge commit" → **Confirm merge**

**Do NOT delete `develop`** — it's the permanent working branch.

### Step 2: Sync Branches

```bash
git checkout main && git pull origin main
git checkout develop && git pull origin develop
git merge main --no-edit && git push origin develop
```

### Step 3: Verify Deployment

Wait for auto-deploy to complete, then:

```bash
# Health check
curl -s [PRODUCTION_URL]/[HEALTH_ENDPOINT]
```

If it fails, check hosting platform logs. See deployment reference doc for troubleshooting.

### Step 4: Smoke Test

```bash
curl -s [PRODUCTION_URL]/[PUBLIC_ENDPOINT] | head -c 200
curl -s -o /dev/null -w "%{http_code}" [PRODUCTION_URL]/
# Expected: 200
```

### Step 5: Security Verification

```bash
# Access control
curl -s -o /dev/null -w "%{http_code}" [PRODUCTION_URL]/[ADMIN_ENDPOINT]
# Expected: 401 or 403

# Security headers
curl -s -I [PRODUCTION_URL]/ | grep -iE 'x-frame-options|x-content-type|strict-transport|content-security'

# No stack traces
curl -s [PRODUCTION_URL]/[NONEXISTENT_ENDPOINT]
# Expected: generic error
```

Record results:
```bash
echo "## Post-Deploy Security Verification — $(date -I)" >> compliance/evidence/REQ-XXX/security-summary.md
echo "- Admin auth check: PASS" >> compliance/evidence/REQ-XXX/security-summary.md
echo "- Security headers: PASS" >> compliance/evidence/REQ-XXX/security-summary.md
echo "- No stack traces: PASS" >> compliance/evidence/REQ-XXX/security-summary.md
```

### Step 6: Finalize Compliance (Tracked Requirements Only)

```bash
mv compliance/pending-releases/RELEASE-TICKET-REQ-XXX.md compliance/approved-releases/
```

Update `compliance/RTM.md`:
```markdown
| REQ-XXX | Description | [RISK] | files | evidence | APPROVED - DEPLOYED | [Reviewer] | [Date] |
```

Add audit trail to release ticket:
```markdown
| [date] | PR approved | [reviewer] | PR #[number] |
| [date] | CI verification | GitHub Actions | All gates passed independently |
| [date] | Deployed to production | System | Auto-deploy from main |
| [date] | Post-deploy verification | [who] | Health + security checks passed |
```

```bash
git add compliance/RTM.md compliance/approved-releases/ compliance/evidence/REQ-XXX/
git rm compliance/pending-releases/RELEASE-TICKET-REQ-XXX.md 2>/dev/null
git commit -m "compliance: [REQ-XXX] approved and deployed - PR #[number]"
git push origin develop
```

### Step 7: Final Sync

```bash
git checkout main && git merge develop --no-edit && git push origin main
git checkout develop
```

## Rollback

1. **Hosting dashboard:** Redeploy previous version
2. **Git:** `git checkout main && git revert HEAD --no-edit && git push origin main`
3. **Document:** Add rollback entry to release ticket audit trail

## Output

- PR merged, deployment verified
- Security verification passed
- Branches synced
- Release ticket finalized
- RTM: `APPROVED - DEPLOYED`

## Pipeline Complete

```
Requirement (RTM + Risk)
  → Test Scope (planned before implementation)
    → AI Use Documented
      → Implementation (develop)
        → Local Gates (SAST + deps + E2E — comprehensive)
          → Evidence Compiled
            → PR Created → CI Gates (independent verification)
              → Human Review (code + CI + evidence + test scope)
                → Deployment (auto-deploy)
                  → Verification (health + security)
                    → Finalization (RTM closed)
```
