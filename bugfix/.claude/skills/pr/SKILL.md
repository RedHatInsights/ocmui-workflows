---
name: pr
description: Create draft pull request
---

# /pr - Create Pull Request

## Purpose

Push the bugfix branch to your fork and create a draft pull request targeting the main uhc-portal repository.

## Prerequisites

- All code changes committed to feature branch
- Tests passing
- PR description prepared
- GitHub authentication configured

## Process

### 1. Pre-flight Checks

Verify readiness:

```bash
cd /workspace/repos/uhc-portal

# Check current branch
git branch --show-current
# Should be: bugfix/OCMUI-{number}-{description}

# Check for uncommitted changes
git status
# Should be clean

# Verify tests pass
yarn test

# Verify linters pass
yarn lint
yarn typecheck
```

If any checks fail, address them before proceeding.

### 2. Ensure Fork Exists

Check if you have a fork configured:

```bash
git remote -v
```

**If you see a fork remote:**
```
origin    https://github.com/RedHatInsights/uhc-portal.git (fetch)
fork      https://github.com/{YOUR-USERNAME}/uhc-portal.git (fetch)
```

**If no fork remote exists:**

Ask the user:
```
I need to push to your fork of uhc-portal. 

Do you have a fork? If not:
1. Go to https://github.com/RedHatInsights/uhc-portal
2. Click "Fork" in the top right
3. Create the fork in your personal account

Once you have a fork, provide the URL:
https://github.com/{YOUR-USERNAME}/uhc-portal.git

I'll add it as a remote named 'fork'.
```

Then add the remote:
```bash
git remote add fork {user-provided-fork-url}
```

### 3. Commit Changes

If there are uncommitted changes:

```bash
git add {modified-files}

git commit -m "fix(OCMUI-{number}): {brief description}

{Detailed explanation from implementation notes}

Fixes: OCMUI-{number}
{If AI assisted: Assisted-by: {tool name}}
"
```

**Commit message format:**
- Type: `fix`
- Scope: `OCMUI-{number}`
- Subject: Brief description (50 chars or less)
- Body: Detailed explanation (wrap at 72 chars)
- Footer: `Fixes: OCMUI-{number}`
- Footer: `Assisted-by: {tool}` (if applicable)

### 4. Push to Fork

```bash
git push -u fork bugfix/OCMUI-{number}-{description}
```

**If push fails** (no permission, no fork access):
- Provide manual instructions
- Give the user the commit details to push themselves

### 5. Create Draft PR

Use GitHub CLI if available:

```bash
gh pr create \
  --repo RedHatInsights/uhc-portal \
  --base main \
  --head {YOUR-USERNAME}:bugfix/OCMUI-{number}-{description} \
  --title "OCMUI-{number}: {brief description}" \
  --body-file artifacts/bugfix/docs/pr-description-{issue-key}.md \
  --draft
```

**If `gh` CLI not available or fails:**

Provide manual instructions:
```
Create a draft PR manually:

1. Go to: https://github.com/RedHatInsights/uhc-portal/compare/main...{YOUR-USERNAME}:bugfix/OCMUI-{number}-{description}

2. Click "Create pull request"

3. Title: OCMUI-{number}: {brief description}

4. Description: Copy content from:
   artifacts/bugfix/docs/pr-description-{issue-key}.md

5. Click the dropdown arrow next to "Create pull request"

6. Select "Create draft pull request"

7. Click "Draft pull request" button
```

### 6. Verify PR Created

If PR creation succeeded:
- Capture the PR URL
- Confirm it's in draft status
- Check that PR title includes Jira issue key

### 7. Generate PR Summary

Create `artifacts/bugfix/docs/pr-summary-{issue-key}.md`:

````markdown
# PR Creation Summary: {Issue Key}

## PR Details

- **URL**: {PR URL}
- **Title**: OCMUI-{number}: {brief description}
- **Base**: main
- **Head**: {YOUR-USERNAME}:bugfix/OCMUI-{number}-{description}
- **Status**: DRAFT

## Branch Information

- **Branch**: bugfix/OCMUI-{number}-{description}
- **Fork**: {fork URL}
- **Commits**: {number of commits}

## Next Steps for User

### 1. Review the Draft PR

Visit: {PR URL}

- Verify the description is complete
- Check that all CI checks are running
- Review the diff to ensure only intended changes are included

### 2. Address CodeRabbit Feedback

CodeRabbit will automatically review the PR. You should:
- Read all CodeRabbit comments
- Address issues or explain why they don't need action
- Resolve threads that CodeRabbit opened

### 3. Assign Reviewers

Once CodeRabbit feedback is addressed:
- Assign 2 developer reviewers
- Add QE reviewer (from Jira QA Contact field, or Jaya if empty)

### 4. Move Jira to Code Review

Update the Jira issue:
- Set status to "Code Review"
- Add comment with PR URL

Use the guidance in: `artifacts/bugfix/docs/jira-updates-{issue-key}.md`

### 5. Mark as Ready for Review

After reviewers are assigned and you're confident in the changes:
- Click "Ready for review" in the PR

### 6. Respond to Review Feedback

When reviewers comment:
- Address all feedback
- Push new commits (don't amend - keep review history)
- If code changes after 2 dev approvals, re-request reviews

### 7. Merge When Approved

After 3 approvals (2 dev + 1 QE):
- Pull latest from main: `git pull origin main`
- Resolve any conflicts
- Wait for all checks to pass
- Click "Squash and merge"
- {If AI assisted: Add AI attribution to commit message}

### 8. Post-Merge Verification

After merging:
- Wait for #ocm-ui-deploys notification
- Test in staging environment
- Update Jira to "Done"

## PR Size Check

- **Lines changed**: {estimate}
- **Files changed**: {count}
- **Within limits**: {yes/no} (max 1000 lines / 30 files)

{If over limits: Recommend discussing with team lead}

## AI Attribution

{If substantial AI contribution:}
⚠️ **Remember**: This PR included substantial AI assistance from {tool name}.

When squashing and merging, add to the commit message:
```
Assisted-by: {tool name}
```

## Reminders

- PR is initially a DRAFT - this lets you review before requesting formal review
- Team requires 2 dev + 1 QE approvals
- CodeRabbit feedback should be addressed first
- Keep PR size under 1000 lines / 30 files when possible
- Test in staging after merge before closing the Jira issue
````

### 8. Re-read Controller and Return

After generating the PR summary, re-read `.claude/skills/controller/SKILL.md` and return control to the controller.

## Output

- PR created in draft status (or manual instructions provided)
- `artifacts/bugfix/docs/pr-summary-{issue-key}.md` — Complete post-PR guide

## Success Criteria

After running this phase:
- [ ] Branch pushed to fork
- [ ] Draft PR created (or manual instructions provided)
- [ ] PR title includes Jira issue key
- [ ] PR description uses team template
- [ ] Next steps documented for user

## Notes

- **Draft PR first**: Creating as draft lets you review before requesting formal review
- **Manual fallback**: If automation fails, provide clear manual instructions
- **Fork requirement**: PRs must come from personal forks, not branches on main repo
- **AI attribution**: Don't forget to include AI tool name if it contributed substantially
- **Post-PR work**: Creating the PR is just the start - guide the user through the full review process
