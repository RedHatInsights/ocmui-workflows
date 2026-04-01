# Bug Fix Workflow for OCMUI/UHC Portal Team

Systematic bug scrubbing and resolution workflow for OCMUI Interruption Catcher duties and general bug fixing. Guides you through bug triage, diagnosis, fix implementation, testing, and PR creation following UHC Portal team standards.

## Overview

This workflow is specifically designed for the OCMUI/UHC Portal team at Red Hat, incorporating:

- **Defect Manager Process**: Bug scrubbing duties from the Interruption Catcher role
- **Team Standards**: UHC Portal coding conventions (TypeScript, React, PatternFly)
- **PR Process**: 2 dev + 1 QE approval workflow
- **Testing Requirements**: Jest/RTL unit tests + Playwright e2e tests + coverage checks

## Quick Start

### Fully Automated Workflow

**`/bugfix-all OCMUI-1234`** - Run the complete workflow automatically without stopping between phases.

Perfect for straightforward bugs where you trust the AI to handle everything end-to-end:
- ✅ Scrubbing and evaluation
- ✅ Root cause analysis
- ✅ Fix implementation
- ✅ Unit tests and coverage
- ✅ Draft PR creation

Use this for simple bugs. For complex or Blocker/Critical bugs, use the manual workflow below.

### Manual Workflow Commands

For step-by-step control:

| Command | Purpose |
|---------|---------|
| `/list-unscrubbed` | List all unscrubbed bugs (Interruption Catcher start) |
| `/scrub OCMUI-1234` | Evaluate a specific bug |
| `/diagnose OCMUI-1234` | Root cause analysis |
| `/fix OCMUI-1234` | Implement the bug fix |
| `/test OCMUI-1234` | Write tests and verify coverage |
| `/draft-pr OCMUI-1234` | Create draft pull request |

## Workflow Phases

### Phase 0: List Unscrubbed Bugs (`/list-unscrubbed`)

**Purpose**: Fetch and display all unscrubbed bugs from OCMUI Jira (INTERRUPTION CATCHER START)

**Process**:
- Executes JQL query to find bugs without 'scrubbed' label
- Displays table with: Key, Priority, Summary, Assignee, Created date, Labels
- Shows summary statistics (total count, priority breakdown)
- Provides guidance on how to scrub a specific bug

**Output**: Console output only (no artifact file)

**When to use**: Start here when beginning Interruption Catcher duty to see what needs scrubbing

⚠️ **Important Limitation**: CVE/Vulnerability tickets are NOT included in `/list-unscrubbed` results due to API security restrictions. You must manually check the Jira dashboard for CVE tickets (look for 🔒 lock icon and "CVE-YYYY-NNNNN" summaries). See [CVE Handling Guide](reference/CVE-HANDLING.md) for detailed instructions.

### Phase 1: Scrub (`/scrub OCMUI-1234`)

**Purpose**: Evaluate a specific bug from the unscrubbed list

**Process**:
- Determine if it's really a bug (vs story/task)
- Check reproducibility
- Identify blockers (needs-uxd, backend dependencies)
- Verify priority level (Blocker → Critical → Major → Normal → Minor)
- Recommend Jira updates (manual for now)
- Add 'scrubbed' label when ready

**Output**: `artifacts/bugfix/scrub-{issue-key}.md`

**Next step**: AI suggests `/diagnose OCMUI-1234` or `/bugfix-all OCMUI-1234`

### Phase 2: Diagnose (`/diagnose OCMUI-1234`)

**Purpose**: Perform root cause analysis

**Process**:
- Locate relevant code in uhc-portal
- Trace execution flow
- Examine git history
- Identify root cause
- Recommend fix strategy

**Output**: `artifacts/bugfix/root-cause-{issue-key}.md`

**Next step**: AI suggests `/fix OCMUI-1234`

### Phase 3: Fix (`/fix OCMUI-1234`)

**Purpose**: Implement the bug fix following UHC Portal standards

**Process**:
- Create feature branch (`bugfix/OCMUI-{number}-{description}`)
- Implement minimal code changes
- Follow TypeScript, React, and PatternFly conventions
- Reference `.cursor/rules/*.mdc` files
- Run linters and formatters

**Output**: Modified code files + `artifacts/bugfix/implementation-{issue-key}.md`

**Next step**: AI suggests `/test OCMUI-1234`

### Phase 4: Test (`/test OCMUI-1234`)

**Purpose**: Verify the fix with comprehensive testing

**Process**:
- Write unit tests (Jest + React Testing Library)
- Run `yarn test-changes` to verify coverage
- Create Playwright e2e tests (when UI changes)
- Follow testing standards from `.cursor/rules/`
- Run full test suite

**Output**: New test files + `artifacts/bugfix/tests-{issue-key}.md`

**Next step**: AI suggests `/draft-pr OCMUI-1234`

### Phase 5: Draft PR (`/draft-pr OCMUI-1234`)

**Purpose**: Create PR description and draft pull request

**Process**:
- Generate PR description using exact team template
- Create Jira update guide
- Push branch to personal fork
- Create draft PR targeting main
- Provide post-PR guidance

**Output**: Draft PR + `artifacts/bugfix/pr-description-{issue-key}.md` + `artifacts/bugfix/jira-updates-{issue-key}.md`

**Next step**: Review PR, update Jira, assign reviewers

## Getting Started

### Prerequisites

- Access to uhc-portal repository
- Access to Jira Defect Manager Dashboard
- Familiarity with the team's PR process
- GitHub authentication configured

### For Interruption Catcher Duty

```bash
# 1. List all unscrubbed bugs
/list-unscrubbed

# 2. Pick a bug and scrub it
/scrub OCMUI-1234

# 3. For simple bugs, use automated workflow
/bugfix-all OCMUI-1234

# OR for complex bugs, go step-by-step
/diagnose OCMUI-1234
/fix OCMUI-1234
/test OCMUI-1234
/draft-pr OCMUI-1234
```

### For Already-Scrubbed Bug

```bash
# Simple bug - use automation
/bugfix-all OCMUI-5678

# Complex bug - go step-by-step
/diagnose OCMUI-5678
# ... continue through phases
```

### For Blocker/Critical Bug

```bash
# Jump to diagnosis (skip scrubbing)
/diagnose OCMUI-9999

# Then continue
/fix OCMUI-9999
/test OCMUI-9999
/draft-pr OCMUI-9999
```

## Example Usage Scenarios

### Scenario 1: Simple Bug - Fully Automated

```bash
User: "Found a simple typo bug OCMUI-1234"

Command: /bugfix-all OCMUI-1234

Result:
✅ Scrub complete - confirmed as bug
✅ Root cause identified - typo in validation message
✅ Fix implemented - corrected spelling
✅ Tests added - regression test + coverage verified
✅ Draft PR created - #456

Next: Review PR and update Jira
```

### Scenario 2: Complex Bug - Manual Workflow

```bash
User: "Complex networking bug OCMUI-5678"

Command: /scrub OCMUI-5678
→ AI: "Next: /diagnose OCMUI-5678"

Command: /diagnose OCMUI-5678
→ AI: "Root cause: race condition. Next: /fix OCMUI-5678"

Command: /fix OCMUI-5678
→ AI: "Fix implemented. Next: /test OCMUI-5678"

Command: /test OCMUI-5678
→ AI: "Tests passing. Next: /draft-pr OCMUI-5678"

Command: /draft-pr OCMUI-5678
→ AI: "Draft PR created: #789"
```

### Scenario 3: Interruption Catcher Scrubbing

```bash
User: "Starting IC duty"

Command: /list-unscrubbed
→ Shows 6 unscrubbed bugs

Command: /scrub OCMUI-1111
→ AI: "Valid bug, Major priority. Next: /bugfix-all OCMUI-1111"

Command: /bugfix-all OCMUI-1111
→ Runs full workflow automatically
```

## UHC Portal Standards

### Code Conventions

- **TypeScript**: Strict typing, proper interfaces, no `any`
- **React**: Functional components, hooks, no inline functions in JSX
- **PatternFly**: Use PatternFly components, no custom CSS
- **Reference**: `.cursor/rules/*.mdc` files in uhc-portal repo

### Testing Requirements

- **Unit Tests**: Jest + React Testing Library
  - Follow Arrange-Act-Assert pattern
  - Test behavior, not implementation
  - Use proper query priorities (`getByRole` first)

- **Coverage**: Run `yarn test-changes` to verify coverage on modified code

- **E2E Tests**: Playwright (when UI changes)
  - Follow page object pattern
  - Use proper test tags (`@ci`, `@smoke`, `@day1`/`@day2`)

### PR Requirements

- **DRAFT first**: Create as draft initially
- **Jira ticket in title**: `OCMUI-XXXX: Brief description`
- **Team template**: Use exact format from `.github/pull_request_template.md`
- **AI attribution**: If substantial AI contribution, note in description and commit
- **Size limits**: Max 1000 lines or 30 files
- **Approvals**: 2 dev + 1 QE required

## Priority Levels

Following OCMUI Defect Manager standards:

- **Blocker**: Complete halt in critical functionality (login fails, white screens)
- **Critical**: Significant degradation, no workaround
- **Major**: Important features affected, complex workaround
- **Normal**: Moderate impact, straightforward workaround
- **Minor**: Minimal impact

## Artifacts Generated

All workflow artifacts in flat structure at `artifacts/bugfix/`:

```
artifacts/bugfix/
├── scrub-OCMUI-1234.md
├── root-cause-OCMUI-1234.md
├── implementation-OCMUI-1234.md
├── tests-OCMUI-1234.md
├── pr-description-OCMUI-1234.md
└── jira-updates-OCMUI-1234.md
```

## CVE/Vulnerability Handling

CVE (Common Vulnerabilities and Exposures) tickets require special handling:

- ⚠️ **Not included in `/list-unscrubbed`** due to API security restrictions
- Manually check Jira dashboard for CVE tickets (🔒 icon)
- Follow the [CVE Handling Guide](reference/CVE-HANDLING.md)
- Use `yarn audit` to scan for vulnerabilities in uhc-portal
- Post investigation findings to the Jira ticket

## Troubleshooting

**Can't reproduce the bug?**
Document what you tried, request more info from reporter via Jira comment

**Bug is blocked (needs UXD input)?**
Set Jira blocked field, add blocked reason and needs-uxd label, wait for unblock

**PR too large (>1000 lines / >30 files)?**
Consider splitting into smaller PRs, or discuss with team lead

**Test coverage is low?**
Add more unit tests focusing on edge cases and error paths

**Not sure if it's a bug or feature request?**
Use `/scrub` phase - it helps determine bug vs story/task

## References

- **Defect Manager Dashboard**: https://issues.redhat.com/secure/Dashboard.jspa?selectPageId=12358493
- **UHC Portal Repo**: https://github.com/RedHatInsights/uhc-portal
- **PR Process**: uhc-portal/docs/pull-request-process.md
- **Coding Rules**: uhc-portal/.cursor/rules/*.mdc
- **CVE Handling**: reference/CVE-HANDLING.md

## Support

For questions or issues:
- #ocm-console-team Slack channel
- Team leads
- Interruption Catcher documentation

---

**Created for**: OCMUI/UHC Portal Team
**Workflow Version**: 2.0.0
**Last Updated**: 2026-04-01
