---
name: reproduce
description: Systematically reproduce bugs in a controlled environment
---

# /reproduce - Bug Reproduction

## Purpose

Systematically reproduce the bug to confirm it exists and document the exact conditions under which it occurs. This establishes a baseline for diagnosis and verification.

## Prerequisites

- Bug report with symptom description
- Access to uhc-portal codebase
- Ability to run the application locally or access staging environment

## Process

### 1. Parse Bug Report

Extract key information:
- **Symptom**: What the user experiences
- **Environment**: Browser, version, user role, cluster type
- **Steps**: Documented reproduction steps
- **Expected**: What should happen
- **Actual**: What actually happens

### 2. Set Up Environment

**For local reproduction:**
- Ensure uhc-portal repo is available
- Run `yarn install` if dependencies need updating
- Start dev server with `yarn start`
- Note the environment (Node version, browser, etc.)

**For staging reproduction:**
- Access staging environment
- Note URL and deployment version

### 3. Attempt Reproduction

Follow the reported steps exactly:
- Document each action taken
- Note what happens at each step
- Capture screenshots or console errors if helpful
- Check browser console for errors
- Check network tab for failed requests

### 4. Vary Conditions

Try variations to understand boundaries:
- Different browsers (Chrome, Firefox, Safari)
- Different user roles or permissions
- Different cluster types (ROSA, OSD, etc.)
- Different data states (empty lists, full lists, edge cases)

Document which variations reproduce the bug and which don't.

### 5. Find Minimal Reproduction

Reduce steps to the minimum required:
- Remove unnecessary navigation
- Identify the critical action that triggers the bug
- Document the simplest path to reproduce

### 6. Assess Severity

Based on reproduction, confirm or adjust priority:
- **Blocker**: Cannot proceed with critical functionality
- **Critical**: Significant impact, no workaround
- **Major**: Important feature affected, complex workaround exists
- **Normal**: Moderate impact, simple workaround exists
- **Minor**: Minimal impact

### 7. Generate Reproduction Report

Create `artifacts/bugfix/reports/reproduction-{issue-key}.md`:

````markdown
# Bug Reproduction Report: {Issue Key}

## Bug Summary

- **Issue**: {OCMUI-XXXX}
- **Title**: {bug title}
- **Reporter**: {name}
- **Environment**: {browser, OS, etc.}

## Reproduction Steps

### Minimal Steps

1. {Step 1}
2. {Step 2}
3. {Step 3}

**Expected Result**: {what should happen}

**Actual Result**: {what actually happens}

### Environment Details

- **Browser**: {Chrome 120, Firefox 115, etc.}
- **Application**: {local dev / staging / production}
- **User Role**: {admin, viewer, etc.}
- **Cluster Type**: {ROSA, OSD-AWS, etc.}

## Reproduction Success

- [ ] Reproduced locally
- [ ] Reproduced on staging
- [ ] Reproduced in multiple browsers
- [ ] Identified minimal steps

## Observations

### Console Errors

```
{paste any console errors}
```

### Network Errors

{any failed API calls, 404s, 500s, etc.}

### Visual Issues

{describe any UI problems: missing elements, misalignment, incorrect styling}

## Variations Tested

| Variation | Result |
|-----------|--------|
| Chrome | {Reproduced / Not reproduced} |
| Firefox | {Reproduced / Not reproduced} |
| Different cluster type | {Reproduced / Not reproduced} |
| Different user role | {Reproduced / Not reproduced} |

## Severity Assessment

**Confirmed Priority**: {Blocker / Critical / Major / Normal / Minor}

**Justification**: {why this priority level is appropriate}

## Unable to Reproduce

{If you cannot reproduce, document:
- What you tried
- What might be different from reporter's environment
- What additional information is needed
}

## Next Steps

{Recommend /diagnose for root cause analysis, or return to /scrub if unable to reproduce}
````

### 8. Re-read Controller and Return

After generating the reproduction report, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- `artifacts/bugfix/reports/reproduction-{issue-key}.md` — Complete reproduction report

## Success Criteria

After running this phase:
- [ ] Bug reproduced (or documented why it couldn't be)
- [ ] Minimal reproduction steps identified
- [ ] Environment conditions documented
- [ ] Severity confirmed or adjusted
- [ ] Variations tested
- [ ] Console/network errors captured

## Notes

- **If you cannot reproduce**: This is valuable information. Document what you tried and recommend asking the reporter for more details or environment specifics.
- **Flaky bugs**: If the bug only reproduces sometimes, document the success rate and any patterns you notice.
- **Environment-specific**: Some bugs only occur in specific environments (production but not staging). Document this clearly.
