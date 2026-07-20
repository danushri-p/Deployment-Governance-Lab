# Deployment Governance Lab

## Learning Objectives

After completing this lab, you will:

- Understand how GitHub Environments protect production deployments
- Configure environment-level deployment approval workflows
- Implement branch restrictions for production deployments
- Scope secrets to specific environments
- Audit deployment governance weaknesses
- Validate that deployments require explicit approval

## Lab Overview

This repository contains a deliberately weak deployment process for the **Checkout Service**, a simple Node.js application.

Your task is to:

1. **Audit** the current deployment pipeline and identify governance gaps
2. **Harden** the GitHub environment with protection rules
3. **Validate** that deployments now require approval before reaching production

The repository simulates a real-world scenario where a team has built a CI/CD pipeline but hasn't properly secured it for production use.

## Course Context

- **Course:** Foundations of Modern Cloud Computing (Semester 7)
- **Lesson:** Deployment Governance and Release Controls
- **Duration:** 60–90 minutes
- **Prerequisites:** GitHub Actions basics, knowledge of branches and CI/CD

## Repository Structure

```
.github/
  └── workflows/
      ├── ci.yml              # Build and test workflow (no deployment)
      └── deploy.yml          # BROKEN deployment workflow
src/
  └── app.js                  # Simple Node.js checkout service
docs/
  ├── audit-guide.md          # Your task checklist
  └── student-notes.md        # Space for your findings
package.json                  # Application dependencies
README.md                     # This file
```

## The Application

This repository contains **Checkout Service**, a simple Node.js service with three endpoints:

```bash
GET  /health          # Health check
POST /checkout        # Process a checkout transaction
GET  /info            # Service information
```

The application itself is **correct and complete**. All problems are in the deployment process, not the code.

## Current State (INTENTIONALLY BROKEN)

The deployment workflow in `.github/workflows/deploy.yml` is deliberately misconfigured:

- Deployments can be triggered from **any branch**
- Deployments occur **immediately without approval**
- **Repository-level secrets** are used for production credentials
- **No wait timer** between approval and execution
- **No required reviewers**
- **No branch restrictions**
- **Overly broad permissions**

This represents a common real-world scenario where security was deferred during initial development.

## Your Tasks

### Task 1 — Audit Deployment Controls (20 minutes)

Examine the current deployment process and document all governance weaknesses.

**What to inspect:**

- Review `.github/workflows/deploy.yml`
  - What branches trigger deployments?
  - Is there an `environment:` field?
  - What permissions does the job have?
  - Where do credentials come from?

- Check GitHub Repository Settings
  - Navigate to Settings → Environments
  - Are any environments configured?
  - What protection rules exist?
  - Are there branch restrictions?

- Review branch protection rules
  - Navigate to Settings → Branches → Branch Protection Rules
  - What rules protect `main`?
  - Are deployments required to be approved?

**Deliverable:**

Document your findings in `docs/student-notes.md`:

```markdown
# Audit Findings

## Weakness 1: [Description]
- Risk: [What could go wrong?]
- Evidence: [Where did you find this in the code/settings?]
- Fix: [How would you address this?]

## Weakness 2: [Description]
...
```

### Task 2 — Harden Production Environment (35 minutes)

Configure GitHub to protect production deployments with approval workflows.

**What to configure:**

1. **Create Production Environment**
   - Navigate to Settings → Environments
   - Click "New environment"
   - Name: `production`

2. **Enable Deployment Branch Rules**
   - Add rule: "Protect deployment branches"
   - Select: "Require branches to be up to date before deployment"
   - Allowed branches: Select `main` only

3. **Configure Required Reviewers**
   - Add: "Require reviewers"
   - Number of reviewers: At least 2
   - Reviewers: Add team members or use a team

4. **Set Wait Timer**
   - Add: "Wait timer"
   - Minutes: 1 (for testing; production typically uses 60)

5. **Configure Environment Secrets**
   - Add environment-scoped secrets:
     - `DEPLOY_KEY`
     - `DATABASE_URL`
     - `API_TOKEN`
   - These are only available to jobs that reference `environment: production`

6. **Update Deployment Workflow**
   - Modify `.github/workflows/deploy.yml`
   - Add `environment: production` to the deploy job
   - This triggers approval workflow

**Verification:**

After configuration, verify:

```bash
# These should show:
# ✓ Production environment exists
# ✓ Deployment branch rules configured
# ✓ 2 required reviewers
# ✓ 1-minute wait timer active
# ✓ Environment secrets populated
```

### Task 3 — Validate Governance (20 minutes)

Trigger a deployment and verify that approval workflow functions correctly.

**Steps:**

1. **Trigger Deployment**
   - Make a small change to `src/app.js` (add a comment)
   - Commit and push to `main`
   - Or use workflow dispatch to trigger `deploy.yml` manually

2. **Observe Workflow Pause**
   - Navigate to Actions tab
   - Watch the deployment job pause at the `environment: production` step
   - Verify that the workflow is **waiting for approval**

3. **Review Approval Prompt**
   - GitHub should prompt reviewers
   - Click "Review deployments"
   - Verify required reviewers are shown

4. **Approve Deployment**
   - As a reviewer, approve the deployment
   - Wait for the wait timer to expire (1 minute)
   - Verify deployment resumes and completes

5. **Verify Logs**
   - Check workflow logs
   - Confirm deployment steps executed
   - Verify success message

**Deliverable:**

Screenshot or describe:

- Workflow paused at approval step
- Approval prompt shown
- Deployment proceeded after approval
- Final deployment success

## Expected Workflow

1. Developer pushes code to `main`
2. CI workflow runs (build, test)
3. Deploy workflow is manually triggered or auto-triggered
4. Workflow pauses at `environment: production` step
5. Required reviewers are notified
6. Workflow waits for approval
7. Reviewer approves deployment
8. Wait timer expires (60 seconds in production)
9. Deployment continues and completes
10. Audit log records who approved and when

## Success Criteria

✅ Production environment is configured in GitHub

✅ Deployment requires explicit approval from 2+ reviewers

✅ Only `main` branch can deploy to production

✅ Production secrets are environment-scoped

✅ Workflow workflow uses `environment: production`

✅ Deployment workflow pauses at approval step

✅ Deployment resumes after approval

✅ Audit trail shows approver information

## Common Questions

**Q: Where do I configure environments?**
A: Settings → Environments (in your GitHub repository)

**Q: How do I see required reviewers?**
A: Settings → Environments → production → "Required reviewers"

**Q: Why doesn't the deployment start immediately?**
A: That's the point! Governance requires approval. This prevents accidental or unauthorized deployments.

**Q: Can I skip the approval for testing?**
A: You can set the wait timer to 0 seconds for testing, but production should use 60+ seconds.

**Q: Where are environment-scoped secrets stored?**
A: Settings → Environments → production → "Environment secrets"

**Q: How is this different from branch protection rules?**
A: Branch protection rules prevent direct pushes to `main`. Environment rules prevent deployments to production environments without approval.

## Troubleshooting

**Problem: Workflow doesn't pause for approval**
- Verify `environment: production` is in the workflow
- Verify production environment exists in Settings → Environments
- Verify you have at least 2 required reviewers configured

**Problem: Can't find "Review deployments" button**
- You must be configured as a required reviewer
- Check Settings → Environments → production → Required reviewers

**Problem: Approval prompt disappeared**
- Approval workflow may have timed out (default 30 days)
- Trigger the workflow again

**Problem: Secrets not accessible in deployment step**
- Verify they're configured in Settings → Environments → production → Environment secrets
- Verify job uses `environment: production`

## Next Steps (After Completing Lab)

1. Document your audit findings in detail
2. Create a governance rubric for your team
3. Review other projects in the organization for similar weaknesses
4. Present findings to senior engineers
5. Implement similar controls in other repositories

## Lab Completion Checklist

- [ ] All weaknesses documented in `docs/student-notes.md`
- [ ] Production environment created and configured
- [ ] Deployment branch rules enforced
- [ ] 2 required reviewers configured
- [ ] Wait timer set (1 minute for testing)
- [ ] Environment-scoped secrets populated
- [ ] Workflow updated with `environment: production`
- [ ] Test deployment triggered and approved
- [ ] Approval workflow validated
- [ ] Screenshots or evidence collected
- [ ] Final report submitted
new changes made a lot
