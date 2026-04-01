---
name: draft-pr
description: Generate PR description and create draft pull request
---

# /draft-pr - Generate Documentation and Create Draft PR

## Purpose

Generate PR description using the UHC Portal template, create Jira update guide, commit changes, push to fork, and create a draft pull request.

## Prerequisites

- Bug fix implemented and tested
- All tests passing
- Linters pass (yarn lint, yarn typecheck)
- GitHub authentication configured

## Process

### 1. Gather Artifact Information

Review the workflow artifacts:
- Scrubbing report (if created)
- Root cause analysis
- Implementation notes
- Test verification report

### 2. Generate PR Description

Create `artifacts/bugfix/pr-description-{issue-key}.md` using the exact UHC Portal template:

```markdown
# Summary

<!-- add a summarized description of the PR content -->
{1-2 sentence summary explaining what this PR fixes}

<!-- add information about AI usage if substantial contributions are generated/assisted by AI tools -->
{If AI assisted: Assisted by: Claude Sonnet 4.5}

# Jira

<!-- link to the corresponding Jira item -->
Fixes [OCMUI-{number}](https://issues.redhat.com/browse/OCMUI-{number})

# Additional information

<!-- any additional information reviewers should know -->

{Concise explanation covering: root cause, fix approach, and testing summary}

# How to Test

<!-- add any useful information for local testing -->

{Step-by-step user flow to verify the fix}

# Screen Captures

| Before                                              | After                                   |
| --------------------------------------------------- | --------------------------------------- |
| <!-- attach a "before" screenshot or video here --> | <!-- attach an "after" capture here --> |

{If no UI changes: "No UI changes - screenshots not applicable"}

# Review process

Please review and follow the [PR process](https://github.com/RedHatInsights/uhc-portal/blob/main/docs/pull-request-process.md).
```

### 3. Generate Jira Updates Guide

Create `artifacts/bugfix/jira-updates-{issue-key}.md`:

```markdown
# Jira Updates: {OCMUI-XXXX}

## When PR Created
- Status: "Code Review"
- Comment: "PR: {url} | {1-line summary}"

## After 2 Dev Approvals
- Status: "Review"
- Comment: "✅ 2 dev approvals, waiting QE"

## After Merge
- Status: "Done"
- Label: `ai-assisted` (if applicable)
- Comment: "✅ Merged in PR #{num} | Deployed to staging {date}"
```

### 4. Pre-flight Checks

Verify readiness:

```bash
cd /workspace/repos/uhc-portal

# Check current branch
git branch --show-current
# Should be: bugfix/OCMUI-{number}-{description}

# Check for uncommitted changes
git status

# Verify tests pass
yarn test

# Verify linters pass
yarn lint
yarn typecheck
```

If any checks fail, address them before proceeding.

### 5. Ensure Fork Exists

Check if you have a fork configured:

```bash
git remote -v
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

### 6. Commit Changes

If there are uncommitted changes:

```bash
git add {modified-files}

git commit -m "fix(OCMUI-{number}): {brief description}

{Detailed explanation from implementation notes}

Fixes: OCMUI-{number}
{If AI assisted: Assisted-by: Claude Sonnet 4.5 <noreply@anthropic.com>}
"
```

**Commit message format:**
- Type: `fix`
- Scope: `OCMUI-{number}`
- Subject: Brief description (50 chars or less)
- Body: Detailed explanation (wrap at 72 chars)
- Footer: `Fixes: OCMUI-{number}`
- Footer: `Assisted-by: Claude Sonnet 4.5` (if applicable)

### 7. Push to Fork

```bash
git push -u fork bugfix/OCMUI-{number}-{description}
```

**If push fails** (no permission, no fork access):
- Provide manual instructions
- Give the user the commit details to push themselves

### 8. Create Draft PR

Use GitHub CLI if available:

```bash
gh pr create \
  --repo RedHatInsights/uhc-portal \
  --base main \
  --head {YOUR-USERNAME}:bugfix/OCMUI-{number}-{description} \
  --title "OCMUI-{number}: {brief description}" \
  --body-file artifacts/bugfix/pr-description-{issue-key}.md \
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
   artifacts/bugfix/pr-description-{issue-key}.md

5. Click the dropdown arrow next to "Create pull request"

6. Select "Create draft pull request"

7. Click "Draft pull request" button
```

### 9. Output Summary

Display next steps concisely:

```markdown
✅ **Draft PR Created**

**Issue:** OCMUI-{XXXX}
**PR:** {URL}
**Branch:** bugfix/OCMUI-{XXXX}-{description}

**Next Steps:**
1. Review PR description and diff
2. Update Jira (see `artifacts/bugfix/jira-updates-OCMUI-{XXXX}.md`)
3. Address CodeRabbit feedback
4. Assign reviewers (2 dev + 1 QE)
5. Mark as "Ready for review"

**After merge:** Test in staging, close OCMUI-{XXXX}
```

### 10. Re-read Controller and Return

After displaying summary, re-read `.claude/skills/controller/SKILL.md` and return control to the controller.

## Output

- `artifacts/bugfix/pr-description-{issue-key}.md` — PR description
- `artifacts/bugfix/jira-updates-{issue-key}.md` — Jira update guide
- PR created in draft status (or manual instructions provided)
- Console output with next steps

## Notes

- **Draft PR first**: Creating as draft lets you review before requesting formal review
- **Manual fallback**: If automation fails, provide clear manual instructions
- **Fork requirement**: PRs must come from personal forks, not branches on main repo
- **AI attribution**: Don't forget to include AI tool name if it contributed substantially
