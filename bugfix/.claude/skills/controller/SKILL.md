---
name: controller
description: Top-level workflow controller that manages phase transitions for OCMUI bug resolution
---

# OCMUI Bugfix Workflow Controller

You are the workflow controller. Your job is to manage the OCMUI bugfix workflow by executing phases and handling transitions between them.

## Automated Workflow

**`/bugfix-all OCMUI-XXXX`** - Run the complete workflow automatically without stopping between phases.

Use this for straightforward bugs where you trust the AI to handle the entire process end-to-end.

## Manual Workflow (Individual Phases)

1. **Scrub** (`/scrub`) — `.claude/skills/scrub/SKILL.md`
   Evaluate unscrubbed bugs from the Defect Manager Dashboard. Determine if it's a bug, check reproducibility, verify priority, identify blockers.

2. **Diagnose** (`/diagnose`) — `.claude/skills/diagnose/SKILL.md`
   Trace the root cause through code analysis, git history, and hypothesis testing.

3. **Fix** (`/fix`) — `.claude/skills/fix/SKILL.md`
   Implement the fix following UHC Portal standards (TypeScript, React, PatternFly).

4. **Test** (`/test`) — `.claude/skills/test/SKILL.md`
   Write unit tests (Jest/RTL), run `yarn test-changes`, create e2e tests (Playwright) if needed.

5. **Draft PR** (`/draft-pr`) — `.claude/skills/draft-pr/SKILL.md`
   Generate PR description, push to fork, and create a draft pull request.

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

**IMPORTANT:** Always include the issue key in recommendations (e.g., `/diagnose OCMUI-4183`) so the user can copy/paste.

### Typical Flow

```text
scrub → diagnose → fix → test → draft-pr
```

Or for simple bugs: `/bugfix-all OCMUI-XXXX`

### What to Recommend

After presenting results, consider what just happened, then offer options that make sense:

**Continuing to the next step** — often the next phase in the flow is the best option

**Skipping forward** — sometimes phases aren't needed:
- Scrub identified a clear bug and root cause → offer `/fix OCMUI-{XXXX}` alongside `/diagnose OCMUI-{XXXX}`
- Bug is already well-documented → skip `/scrub`, go to `/diagnose OCMUI-{XXXX}`
- Blocker/Critical bug → offer `/diagnose OCMUI-{XXXX}` or `/fix OCMUI-{XXXX}` immediately

**Going back** — sometimes earlier work needs revision:
- Test failures → offer `/fix OCMUI-{XXXX}` to rework the implementation
- Can't reproduce → go back to `/scrub OCMUI-{XXXX}` to reassess
- Diagnosis was wrong → offer `/diagnose OCMUI-{XXXX}` again with new information

**Ending early** — not every bug needs the full pipeline:
- A trivial fix might go straight from `/fix` → `/test` → `/draft-pr`
- If the user already has their own PR process, they may stop after `/test`

**Going fully automated** — for straightforward bugs:
- If bug is simple and clear → offer `/bugfix-all OCMUI-{XXXX}` to automate the rest

### How to Present Options

Lead with your top recommendation, then list alternatives briefly. **Always include the issue key:**

```text
Recommended next step: /test OCMUI-4183 — verify the fix with unit tests and coverage check.

Other options:
- /draft-pr OCMUI-4183 — if you've already tested manually and want to create the PR
- /bugfix-all OCMUI-4183 — automate the remaining phases (test + draft-pr)
```

### Special Cases for OCMUI Workflow

**After /scrub:**
- If bug is Blocker/Critical → recommend `/diagnose OCMUI-{XXXX}` or `/fix OCMUI-{XXXX}` immediately
- If bug cannot be reproduced → recommend adding comment to Jira asking for more info
- If bug is actually a story/task → recommend changing Jira type
- If bug is blocked → recommend setting blocked field and reason in Jira
- If bug is simple and straightforward → offer `/bugfix-all OCMUI-{XXXX}` to automate everything
- **Always include issue key in next step:** `/diagnose OCMUI-{XXXX}`

**After /diagnose:**
- For OCM API-related bugs → recommend validating API contracts with `ocm-api-model` repo
- For PatternFly UI component bugs → recommend checking PatternFly docs via Context7 MCP
- Always recommend `/fix OCMUI-{XXXX}` to implement the identified solution
- If fix is trivial → offer `/bugfix-all OCMUI-{XXXX}` to automate fix + test + PR
- **Always include issue key**

**After /fix:**
- Always recommend `/test OCMUI-{XXXX}` to run `yarn test-changes` and verify coverage
- Remind about PR size limits (1000 lines / 30 files)
- **Always include issue key**

**After /test:**
- If coverage is insufficient → recommend going back to `/test OCMUI-{XXXX}` to add more tests
- If tests pass → recommend `/draft-pr OCMUI-{XXXX}` to create the PR
- **Always include issue key**

**After /draft-pr:**
- Remind to update Jira (see jira-updates file)
- Remind to address CodeRabbit feedback
- Remind about 2 dev + 1 QE approval requirement
- Reference the issue key when discussing Jira updates
