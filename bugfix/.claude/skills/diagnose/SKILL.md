---
name: diagnose
description: Root cause analysis for bugs
---

# /diagnose - Root Cause Analysis

## Purpose

Perform systematic root cause analysis to understand WHY the bug occurs, not just WHAT happens. This guides the fix implementation.

## Prerequisites

- Bug has been scrubbed
- Access to uhc-portal codebase
- Understanding of the reported issue

## Process

### 1. Review Bug Information

Understand from Jira or scrub report:
- Exact steps to trigger the bug
- Observable symptoms
- Expected vs actual behavior
- Environment conditions

### 2. Locate Relevant Code

Use file navigation to find the affected code:
- Search for component names from the bug (use Grep)
- Look for error messages in the codebase (use Grep)
- Find files related to the feature area (use Glob)
- Check routing to understand page flow

### 3. Trace Execution Flow

Follow the code path:
- Identify the UI component where the bug manifests
- Trace data flow: props → state → API calls
- Check Redux/React Query state management
- Follow event handlers and callbacks
- Examine form validation logic if applicable

### 4. Examine Git History

Understand recent changes:
```bash
git log --oneline --since="30 days ago" -- {affected-file}
git blame {affected-file}
```

Look for:
- Recent changes to the affected code
- Related PRs that might have introduced the bug
- Commits around the time the bug was first reported

### 5. Identify Root Cause

Test hypotheses:
- Add console.log or debugger statements
- Run the reproduction steps
- Examine the actual values and execution flow
- Confirm the actual root cause

### 6. Recommend Fix Strategy

Based on the root cause, propose how to fix it:
- **Minimal change**: What's the smallest fix?
- **Correct approach**: What's the right way to solve this?
- **Breaking changes**: Will this affect existing behavior?

### 7. Generate Root Cause Analysis

Create `artifacts/bugfix/root-cause-{issue-key}.md`:

```markdown
# Root Cause: {OCMUI-XXXX}

## Cause
{1-2 sentence statement of the problem}

## Affected Code
- `{file}:{line}` - {issue description}

## Fix Strategy
{Approach to fix}

**Changes needed:**
- {file} - {what needs to change}

**Confidence:** High/Medium/Low - {why}

## Next Step
**Issue:** OCMUI-{XXXX}
**Ready for:** /fix OCMUI-{XXXX}
```

### 8. Re-read Controller and Return

After generating the root cause analysis, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- `artifacts/bugfix/root-cause-{issue-key}.md` — Root cause analysis
- Console output with next step: `/fix {issue-key}`

## Notes

- **Use file:line notation**: Always reference code locations as `src/components/ClusterList.tsx:245`
- **Test hypotheses systematically**: Don't just guess - add logging and verify
- **Be thorough**: Root cause analysis quality directly affects fix quality
