---
name: bugfix-all
description: Fully automated bugfix workflow from scrub to draft PR
---

# /bugfix-all - Automated End-to-End Bugfix

## Purpose

Run the complete bugfix workflow automatically without stopping between phases. This is ideal for straightforward bugs where you trust the AI to handle the entire process end-to-end.

## Usage

```bash
/bugfix-all OCMUI-1234
```

Or with a Jira URL:
```bash
/bugfix-all https://issues.redhat.com/browse/OCMUI-1234
```

## What It Does

Executes the full workflow automatically:

```
/scrub → /diagnose → /fix → /test → /draft-pr
```

**Automated steps:**
1. **Scrub** - Evaluate the bug, verify priority, check reproducibility
2. **Diagnose** - Identify root cause through code analysis
3. **Fix** - Implement the fix following UHC Portal standards
4. **Test** - Write unit tests and verify coverage
5. **Draft PR** - Generate PR description and create draft pull request

## Prerequisites

- Jira issue key or URL
- GitHub authentication configured (GITHUB_TOKEN)
- Access to uhc-portal codebase

## Process

### 1. Parse Issue Key

Extract issue key from argument:
- `OCMUI-1234` → use directly
- `https://issues.redhat.com/browse/OCMUI-1234` → extract key
- If invalid format, show error and exit

### 2. Execute Workflow Phases

For each phase, read the skill file and execute it:

```bash
# Phase 1: Scrub
Read `.claude/skills/scrub/SKILL.md`
Execute scrubbing process
Generate artifacts/bugfix/scrub-{key}.md

# Phase 2: Diagnose
Read `.claude/skills/diagnose/SKILL.md`
Execute root cause analysis
Generate artifacts/bugfix/root-cause-{key}.md

# Phase 3: Fix
Read `.claude/skills/fix/SKILL.md`
Implement the fix
Generate artifacts/bugfix/implementation-{key}.md

# Phase 4: Test
Read `.claude/skills/test/SKILL.md`
Write unit tests, run coverage
Generate artifacts/bugfix/tests-{key}.md

# Phase 5: Draft PR
Read `.claude/skills/draft-pr/SKILL.md`
Generate PR description, push branch, create PR
Generate artifacts/bugfix/pr-description-{key}.md
Generate artifacts/bugfix/jira-updates-{key}.md
```

### 3. Handle Errors

If any phase fails:
- Stop the workflow
- Report which phase failed
- Show the error
- Provide guidance on how to continue manually

**Example:**
```
❌ Workflow stopped at /diagnose phase

Error: Could not identify root cause - multiple potential issues found

To continue manually:
1. Review artifacts/bugfix/root-cause-OCMUI-1234.md
2. Investigate the code manually
3. Run: /fix OCMUI-1234
```

### 4. Output Summary

At the end, provide a complete summary:

```markdown
✅ **Automated Bugfix Complete**

**Issue:** OCMUI-{XXXX}
**PR:** {URL}
**Branch:** bugfix/OCMUI-{XXXX}-{description}

## Artifacts Generated
- scrub-OCMUI-{XXXX}.md
- root-cause-OCMUI-{XXXX}.md
- implementation-OCMUI-{XXXX}.md
- tests-OCMUI-{XXXX}.md
- pr-description-OCMUI-{XXXX}.md
- jira-updates-OCMUI-{XXXX}.md

## Next Steps
1. Review PR: {URL}
2. Update Jira: See artifacts/bugfix/jira-updates-OCMUI-{XXXX}.md
3. Address CodeRabbit feedback
4. Assign reviewers (2 dev + 1 QE)
5. Mark as "Ready for review"

**After merge:** Test in staging, close OCMUI-{XXXX}
```

## When to Use

**Good for:**
- ✅ Straightforward bugs with clear reproduction
- ✅ Bugs with obvious root causes
- ✅ Minor/Normal priority bugs
- ✅ When you trust the AI to handle the full process

**Not recommended for:**
- ❌ Blocker/Critical bugs (review each step manually)
- ❌ Complex bugs with multiple root causes
- ❌ Bugs requiring architectural changes
- ❌ Bugs with unclear reproduction steps

## Manual Intervention

If you want more control, use individual phases instead:
- `/scrub OCMUI-1234` - Evaluate the bug
- `/diagnose OCMUI-1234` - Root cause analysis
- `/fix OCMUI-1234` - Implement fix
- `/test OCMUI-1234` - Write tests
- `/draft-pr OCMUI-1234` - Create PR

## Output

- All artifact files in `/workspace/artifacts/bugfix/`
- Draft PR created on GitHub
- Console summary with next steps

## Notes

- **Fully automated**: No user input required between phases
- **Error handling**: Stops if any phase fails
- **All artifacts generated**: Same as running each phase manually
- **Draft PR**: Always created as draft for review
- **AI attribution**: Automatically included in commit and PR
