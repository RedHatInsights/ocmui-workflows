---
name: scrub
description: Bug scrubbing and triage for OCMUI Interruption Catcher duties
---

# /scrub - Bug Scrubbing and Triage

## Purpose

Evaluate unscrubbed bugs from the OCMUI Defect Manager Dashboard to determine if they're ready to be worked on. This is the primary entry point for Interruption Catcher duties.

## Prerequisites

- Access to Jira Defect Manager Dashboard: https://issues.redhat.com/secure/Dashboard.jspa?selectPageId=12358493
- Bug report or Jira issue URL to scrub

## Process

### 1. Load Bug Information

- If given a Jira URL, extract the issue key (e.g., OCMUI-1234)
- If given a description, ask for the Jira URL
- Note: actual Jira API integration coming later - for now, work with information the user provides

### 2. Evaluate: Is This Really a Bug?

**Bugs are broken functionality.** Ask yourself:

- Does this describe functionality that doesn't work as designed?
- OR is this a suggestion/recommendation/improvement? (→ should be Story or Task)

**If it's NOT a bug:**
- Recommend changing Jira type to "Story" or "Task"
- Explain why (e.g., "This appears to be a feature request rather than broken functionality")

### 3. Check Reproducibility

**Can the bug be reproduced?**

- Review the bug description for reproduction steps
- Check if enough information is provided:
  - Environment (browser, version, etc.)
  - Steps to reproduce
  - Expected vs actual behavior
- Assess likelihood of reproduction

**If information is missing:**
- List what information is needed
- Recommend adding a comment to Jira requesting clarification from the reporter

**If it appears unreproducible:**
- Recommend bringing up in post-scrum for team discussion
- Note: if no response within 3 weeks → candidate for closure

### 4. Identify Blockers

**Is the bug blocked?**

Check if any of these apply:
- **needs-uxd**: Waiting for UX design input
- **Backend dependencies**: Requires backend API changes or information
- **Missing information**: Cannot proceed without clarification

**If blocked:**
- Recommend setting "Blocked" field to "true"
- Recommend adding specific reason to "Blocked Reason" field
- If needs-uxd, recommend adding "needs-uxd" label

### 5. Verify Priority Level

Apply the OCMUI priority criteria:

**Blocker** - Complete halt in critical functionality
- Examples: Login fails, white screens on critical pages, form submission fails with no error
- Action: Defect manager attempts fix immediately

**Critical** - Significant degradation, no workaround
- Examples: Unexpected errors shown to user, critical functionality unavailable
- Action: Work on immediately after in-progress work completes

**Major** - Important features affected, complex workaround
- Examples: Missing/incorrect data, validation prevents valid input, keyboard navigation broken
- Action: Schedule for next sprint

**Normal** - Moderate impact, straightforward workaround
- Examples: Incorrect defaults (but user can change), misspellings, vague labels
- Action: Schedule in next or future sprint

**Minor** - Minimal impact
- Examples: Misaligned buttons, broken help links
- Action: Schedule for future sprint

**If priority seems incorrect:**
- Recommend the correct priority level with justification

### 6. Assignment Decision

**Should anyone be assigned?**

Only assign if:
- Developer has special knowledge of the area AND
- Developer has confirmed capacity in next sprint

OR

- Bug is tied to a parent story AND
- Priority is Blocker, Critical, or Major
  - Assign to developer on parent story
  - Add to current sprint

**Otherwise:**
- Recommend leaving unassigned
- If someone has relevant knowledge, recommend adding a comment tagging them

### 7. Generate Scrubbing Report

Create `artifacts/bugfix/scrubbing/scrub-report-{issue-key}.md` with:

**Bug Summary:**
- Issue: {issue-key}
- Title: {bug title}
- Current Priority: {current}
- Reported by: {reporter}

**Scrubbing Analysis:**

✅ **Type Check:** [Bug / Story / Task]
- Reasoning: {why}

✅ **Reproducibility:** [Can Reproduce / Needs Info / Unreproducible]
- Status: {assessment}
- Missing info: {list if applicable}

✅ **Blockers:** [Blocked / Not Blocked]
- Type: {needs-uxd / backend / other}
- Reason: {specific blocker}

✅ **Priority Assessment:** [Blocker / Critical / Major / Normal / Minor]
- Current: {current priority}
- Recommended: {recommended priority}
- Justification: {why}

✅ **Assignment:** [Assign to X / Leave Unassigned]
- Recommendation: {who and why, or why unassigned}

**Recommended Jira Updates:**

```text
[ ] Change type to: {Story/Task} (if not a bug)
[ ] Set Priority to: {recommended level}
[ ] Set Blocked field to: {true/false}
[ ] Add Blocked Reason: {reason}
[ ] Add label: needs-uxd (if applicable)
[ ] Add label: scrubbed
[ ] Assign to: {developer name} (if applicable)
[ ] Add to sprint: {current sprint name} (if Blocker/Critical with parent story)
[ ] Add comment requesting: {missing information}
```

**Next Steps:**

{Recommend /reproduce, /diagnose, /fix, or waiting for more info}

### 8. Re-read Controller and Return

After generating the scrub report, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- `artifacts/bugfix/scrubbing/scrub-report-{issue-key}.md` — Complete scrubbing analysis and Jira update recommendations

## Success Criteria

After running this phase:
- [ ] Determined if issue is truly a bug
- [ ] Assessed reproducibility
- [ ] Identified any blockers
- [ ] Verified or corrected priority level
- [ ] Made assignment recommendation
- [ ] Generated checklist of Jira updates to perform manually
- [ ] Recommended next workflow phase

## Notes

- **Blocker/Critical bugs**: If you identify a Blocker or Critical bug, recommend jumping to `/diagnose` or `/fix` immediately
- **Manual Jira updates**: For now, all Jira updates are manual - we provide recommendations in the scrub report
- **Scrubbed label**: Only recommend adding 'scrubbed' label if bug is ready to be worked on (enough info, not blocked, priority correct)
- **3-week rule**: If additional info requested and not provided within 3 weeks, bug is candidate for closure
