---
name: diagnose
description: Root cause analysis for bugs
---

# /diagnose - Root Cause Analysis

## Purpose

Perform systematic root cause analysis to understand WHY the bug occurs, not just WHAT happens. This guides the fix implementation.

## Prerequisites

- Reproduction report showing how to trigger the bug
- Access to uhc-portal codebase
- Ability to run the application locally

## Process

### 1. Review Reproduction Report

Load the reproduction report to understand:
- Exact steps to trigger the bug
- Observable symptoms
- Console/network errors
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

### 5. Form Hypotheses

Based on code analysis, propose potential root causes:

**Example hypotheses:**
- "Missing null check when data is undefined"
- "Race condition between API call and state update"
- "Incorrect Redux selector causing stale data"
- "Form validation rule too restrictive"
- "Missing error handling for API failure"

### 6. Test Hypotheses

For each hypothesis:
- Add console.log or debugger statements
- Run the reproduction steps
- Examine the actual values and execution flow
- Confirm or refute the hypothesis

Continue until you identify the actual root cause.

### 7. Assess Impact

Consider broader implications:
- Are there similar patterns elsewhere in the codebase?
- Does this affect other features or pages?
- Are there related bugs that might have the same root cause?

Use Grep to search for similar code patterns.

### 8. Recommend Fix Strategy

Based on the root cause, propose how to fix it:
- **Minimal change**: What's the smallest fix?
- **Correct approach**: What's the right way to solve this?
- **Similar issues**: Should we fix related patterns?
- **Breaking changes**: Will this affect existing behavior?

### 9. Generate Root Cause Analysis

Create `artifacts/bugfix/analysis/root-cause-{issue-key}.md`:

````markdown
# Root Cause Analysis: {Issue Key}

## Bug Summary

- **Issue**: {OCMUI-XXXX}
- **Title**: {bug title}
- **Affected Component**: {component name or page}

## Reproduction Summary

{Brief summary from reproduction report}

## Code Investigation

### Affected Files

- `{file-path}:{line}` — {description}
- `{file-path}:{line}` — {description}

### Execution Flow

1. User action: {action}
2. Event handler: `{function-name}` at `{file}:{line}`
3. State update: {describe}
4. API call (if applicable): `{endpoint}`
5. Error occurs: {where and why}

### Git History

Recent changes to affected area:
```
{relevant git log output}
```

**Relevant PR**: {PR link if identified}

## Root Cause

**Cause**: {concise statement of the actual problem}

**Details**: {detailed explanation of why this causes the observed behavior}

**Code Evidence**:
```typescript
// At {file}:{line}
{paste the problematic code}
```

## Hypotheses Tested

1. ❌ **{Hypothesis 1}**: Ruled out because {reason}
2. ❌ **{Hypothesis 2}**: Ruled out because {reason}
3. ✅ **{Actual cause}**: Confirmed by {evidence}

## Impact Assessment

### Scope

- **Affected pages**: {list}
- **Affected features**: {list}
- **User impact**: {who is affected and how}

### Similar Patterns

{Use Grep to find similar code patterns}

Found {N} similar occurrences in:
- `{file}:{line}` — {should this be fixed too?}
- `{file}:{line}` — {should this be fixed too?}

## Recommended Fix Strategy

### Approach

{Describe the fix strategy}

### Implementation

**Files to modify:**
- `{file}` — {what to change}
- `{file}` — {what to change}

**Changes required:**
1. {Change 1}
2. {Change 2}
3. {Change 3}

### Considerations

- **Breaking changes**: {yes/no and why}
- **TypeScript updates**: {any type changes needed}
- **Testing needs**: {what tests to add/modify}
- **Similar issues**: {should we fix related patterns?}

### Confidence Level

**High** / **Medium** / **Low**

{Explain confidence level}

## References

- File paths: Use `{file}:{line}` notation
- Related issues: {JIRA keys}
- Related PRs: {PR links}

## Next Steps

Ready to proceed to `/fix` phase.
````

### 10. Re-read Controller and Return

After generating the root cause analysis, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- `artifacts/bugfix/analysis/root-cause-{issue-key}.md` — Complete root cause analysis

## Success Criteria

After running this phase:
- [ ] Root cause identified and verified
- [ ] Code paths traced
- [ ] Git history reviewed
- [ ] Impact assessed
- [ ] Fix strategy recommended
- [ ] Confidence level stated

## Notes

- **Use file:line notation**: Always reference code locations as `src/components/ClusterList.tsx:245`
- **Test hypotheses systematically**: Don't just guess - add logging and verify
- **Consider similar patterns**: Use Grep to find related code that might have the same issue
- **Be thorough**: Root cause analysis quality directly affects fix quality
