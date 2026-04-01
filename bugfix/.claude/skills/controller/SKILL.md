---
name: controller
description: Top-level workflow controller that manages phase transitions for OCMUI bug resolution
---

# OCMUI Bugfix Workflow Controller

You are the workflow controller. Your job is to manage the OCMUI bugfix workflow by executing phases and handling transitions between them.

## Phases

1. **Scrub** (`/scrub`) — `.claude/skills/scrub/SKILL.md`
   Evaluate unscrubbed bugs from the Defect Manager Dashboard. Determine if it's a bug, check reproducibility, verify priority, identify blockers.

2. **Reproduce** (`/reproduce`) — `.claude/skills/reproduce/SKILL.md`
   Systematically reproduce the bug in a controlled environment.

3. **Diagnose** (`/diagnose`) — `.claude/skills/diagnose/SKILL.md`
   Trace the root cause through code analysis, git history, and hypothesis testing.

4. **Fix** (`/fix`) — `.claude/skills/fix/SKILL.md`
   Implement the fix following UHC Portal standards (TypeScript, React, PatternFly).

5. **Test** (`/test`) — `.claude/skills/test/SKILL.md`
   Write unit tests (Jest/RTL), run `yarn test-changes`, create e2e tests (Playwright) if needed.

6. **Document** (`/document`) — `.claude/skills/document/SKILL.md`
   Create PR description using team template, recommend Jira updates.

7. **PR** (`/pr`) — `.claude/skills/pr/SKILL.md`
   Push to fork and create a draft pull request.

Phases can be skipped or reordered at the user's discretion.

## How to Execute a Phase

1. **Announce** the phase to the user before doing anything else. Include this file as the dispatcher so skills know where to return, e.g., "Starting the /scrub phase (dispatched by `.claude/skills/controller/SKILL.md`)." This is very important so the user knows the workflow is working, learns about the available phases, and so skills can find their way back here.
2. **Read** the skill file from the list above
3. **Execute** the skill's steps directly — the user should see your progress
4. When the skill is done, it will report its findings and re-read this controller. Then use "Recommending Next Steps" below to offer options.
5. Present the skill's results and your recommendations to the user
6. **Stop and wait** for the user to tell you what to do next

## Recommending Next Steps

After each phase completes, present the user with **options** — not just one next step. Use the typical flow as a baseline, but adapt to what actually happened.

### Typical Flow

```text
scrub → reproduce → diagnose → fix → test → document → pr
```

### What to Recommend

After presenting results, consider what just happened, then offer options that make sense:

**Continuing to the next step** — often the next phase in the flow is the best option

**Skipping forward** — sometimes phases aren't needed:
- Scrub identified a clear bug and root cause → offer `/fix` alongside `/reproduce`
- Bug is already well-documented → skip `/scrub`, go to `/reproduce`
- Blocker/Critical bug → offer `/diagnose` or `/fix` immediately

**Going back** — sometimes earlier work needs revision:
- Test failures → offer `/fix` to rework the implementation
- Can't reproduce → go back to `/scrub` to reassess
- Diagnosis was wrong → offer `/diagnose` again with new information

**Ending early** — not every bug needs the full pipeline:
- A trivial fix might go straight from `/fix` → `/test` → `/pr`
- If the user already has their own PR process, they may stop after `/test`

### How to Present Options

Lead with your top recommendation, then list alternatives briefly:

```text
Recommended next step: /test — verify the fix with unit tests and coverage check.

Other options:
- /document — if you've already tested manually and want to prepare the PR
- /pr — if you're confident and want to submit immediately
```

### Special Cases for OCMUI Workflow

**After /scrub:**
- If bug is Blocker/Critical → recommend `/diagnose` or `/fix` immediately
- If bug cannot be reproduced → recommend adding comment to Jira asking for more info
- If bug is actually a story/task → recommend changing Jira type
- If bug is blocked → recommend setting blocked field and reason in Jira

**After /fix:**
- Always recommend `/test` to run `yarn test-changes` and verify coverage
- Remind about PR size limits (1000 lines / 30 files)

**After /test:**
- If coverage is insufficient → recommend going back to `/test` to add more tests
- If tests pass → recommend `/document` to prepare PR description

**After /document:**
- Remind to create DRAFT PR initially
- Remind about 2 dev + 1 QE approval requirement
