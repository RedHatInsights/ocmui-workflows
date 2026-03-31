---
name: document
description: Create PR description and recommend Jira updates
---

# /document - Documentation

## Purpose

Create a PR description using the UHC Portal team's template and recommend Jira updates to track the bug fix through the review process.

## Prerequisites

- Bug fix implemented and tested
- Test verification report available
- Understanding of the fix and its impact

## Process

### 1. Gather Artifact Information

Review the workflow artifacts:
- Scrubbing report (if created)
- Reproduction report
- Root cause analysis
- Implementation notes
- Test verification report

### 2. Create PR Description

Use the UHC Portal PR template structure (from `.github/pull_request_template.md`):

Create `artifacts/bugfix/docs/pr-description-{issue-key}.md`:

````markdown
# Summary

{1-2 sentence summary of what the PR fixes}

{If substantial AI contribution:}
Assisted by: {AI tool name, e.g., "Cursor/gemini-2.5-pro"}

# Jira

Fixes [OCMUI-{number}](https://issues.redhat.com/browse/OCMUI-{number})

# Additional information

## Root Cause

{Brief explanation of what caused the bug - reference root-cause.md}

## Fix Approach

{Explain how the fix works - reference implementation.md}

## Testing

{Summarize testing done - reference verification.md}
- Unit tests added: {list key tests}
- Coverage: {percentage from yarn test-changes}
- Manual verification: {what you tested}
- E2E tests: {added/updated/not needed}

## Files Modified

- `{file}:{lines}` — {what changed}
- `{file}:{lines}` — {what changed}

# How to Test

1. Check out this branch
2. Run `yarn install` and `yarn start`
3. {Step-by-step instructions to verify the fix}
4. Verify that {expected behavior}

# Screen Captures

| Before | After |
| ------ | ----- |
| {Screenshot showing the bug} | {Screenshot showing it fixed} |

{Note if screenshots aren't applicable}

# Review process

Please review and follow the [PR process](https://github.com/RedHatInsights/uhc-portal/blob/main/docs/pull-request-process.md).

## QE Reviewer

- [ ] _Pre-merge testing : Verified change locally in a browser (downloaded and ran code using reviewx tool)_
- [ ] Updated/created Polarion test cases which were peer QE reviewed
- [ ] Confirmed 'tc-approved' label was added by dev to the linked JIRA ticket
- [ ] (optional) Updated/created Cypress e2e tests
- [ ] Closed threads I started after the author made changes or added an explanation
````

### 3. Recommend Jira Updates

Create `artifacts/bugfix/docs/jira-updates-{issue-key}.md`:

````markdown
# Jira Update Recommendations: {Issue Key}

## When PR is Created

**Set status to**: Code Review

**Add comment**:
```
PR created: {PR URL}

Summary: {brief summary of the fix}

Please review following the team's PR process:
- 2 developer reviews required
- 1 QE review required
- CodeRabbit feedback should be addressed

Branch: bugfix/OCMUI-{number}-{description}
```

## During Code Review

**Respond to CodeRabbit**:
- Address all CodeRabbit comments or explain why they don't need action
- Resolve threads opened by CodeRabbit

**Assign reviewers**:
- Add 2 developer reviewers
- Add QE reviewer (from QA Contact field, or Jaya if empty)

**If code changes after 2 dev approvals**:
- Re-request reviews from both developers

## After 2 Dev Approvals

**Set status to**: Review

**Add comment**:
```
✅ 2 developer approvals received
Waiting for QE review
```

## After QE Approval

**Before merging**:
- [ ] Ensure 3 approvals (2 dev + 1 QE)
- [ ] Pull latest from main
- [ ] Verify all checks pass
- [ ] Verify no merge conflicts

**When merging**:
- Use "Squash and merge" button
- {If AI contributed: Add AI attribution to commit message}

**After merge**:
- [ ] Wait for deployment to staging (check #ocm-ui-deploys)
- [ ] Test in staging environment
- [ ] Verify fix works in staging

## Close the Issue

**Set status to**: Done

**Add comment**:
```
✅ Fixed in PR: {PR URL}
✅ Merged to main: {commit SHA}
✅ Deployed to staging: {date}
✅ Verified in staging

Fix summary: {one sentence}
```

**If substantial AI contribution:**
Add label: `ai-assisted`

## Fields to Update

**Throughout the process:**
- **Status**: To Do → Code Review → Review → Done
- **Assignee**: Keep assigned to you
- **Sprint**: Should be in current sprint
- **Labels**: Maintain `scrubbed` label, add `ai-assisted` if applicable

## PR Size Note

{If PR is large:}
⚠️ **PR Size**: This PR modifies {N} lines across {M} files.
- Team limit: 1000 lines / 30 files
- {Within limits / Over limits}

{If over limits:}
Consider discussing with team lead about splitting the PR.
````

### 4. Create Issue Update

Create `artifacts/bugfix/docs/issue-update-{issue-key}.md`:

````markdown
# Issue Update: {Issue Key}

## Summary Comment for Jira

**Post this comment when work begins:**

```
🔧 Working on this bug

**Root Cause**: {one sentence}

**Fix**: {one sentence}

**Branch**: bugfix/OCMUI-{number}-{description}

**Status**: In progress
```

## Detailed Technical Comment (Optional)

{If the fix is complex or non-obvious, add this technical detail:}

```
**Technical Details**

**Root Cause**:
{Detailed explanation from root-cause.md}

**Fix Approach**:
{Detailed explanation from implementation.md}

**Files Modified**:
- {file} — {change}
- {file} — {change}

**Testing**:
- Unit tests: {summary}
- Coverage: {percentage}
- Manual verification: {summary}
```
````

### 5. Re-read Controller and Return

After generating all documentation, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- `artifacts/bugfix/docs/pr-description-{issue-key}.md` — Complete PR description
- `artifacts/bugfix/docs/jira-updates-{issue-key}.md` — Step-by-step Jira update guide
- `artifacts/bugfix/docs/issue-update-{issue-key}.md` — Comments to post on Jira

## Success Criteria

After running this phase:
- [ ] PR description created using team template
- [ ] AI attribution included if applicable
- [ ] How to Test section provides clear steps
- [ ] Jira update recommendations documented
- [ ] Issue update comments prepared

## Notes

- **Use the team template**: The PR template is standard across the team - follow it exactly
- **AI attribution is important**: If AI contributed substantially, include it in both PR description and commit message
- **QE reviewer checklist**: Don't modify the QE section - they fill it out
- **Jira workflow**: The status transitions are important for team metrics and process
- **Screen captures**: If possible, add before/after screenshots - they help reviewers understand the impact
- **Manual Jira updates**: For now, all Jira updates are manual - we provide step-by-step guidance
