# Bug Fix Workflow for OCMUI/UHC Portal Team

Systematic bug scrubbing and resolution workflow for OCMUI Interruption Catcher duties and general bug fixing. Guides you through bug triage, reproduction, diagnosis, fix implementation, testing, and PR creation following UHC Portal team standards.

## Overview

This workflow is specifically designed for the OCMUI/UHC Portal team at Red Hat, incorporating:

- **Defect Manager Process**: Bug scrubbing duties from the Interruption Catcher role
- **Team Standards**: UHC Portal coding conventions (TypeScript, React, PatternFly)
- **PR Process**: 2 dev + 1 QE approval workflow
- **Testing Requirements**: Jest/RTL unit tests + Playwright e2e tests + coverage checks

## Workflow Commands

### List Unscrubbed Bugs (`/list-unscrubbed`) - INTERRUPTION CATCHER START

**Purpose**: Fetch and display all unscrubbed bugs from OCMUI Jira

**Process**:
- Executes JQL query to find bugs without 'scrubbed' label
- Displays table with: Key, Priority, Summary, Assignee, Created date, Labels
- Shows summary statistics (total count, priority breakdown)
- Provides guidance on how to scrub a specific bug

**Output**: Console output only (no artifact file)

**When to use**: Start here when beginning Interruption Catcher duty to see what needs scrubbing

⚠️ **Important Limitation**: CVE/Vulnerability tickets are NOT included in `/list-unscrubbed` results due to API security restrictions. You must manually check the Jira dashboard for CVE tickets (look for 🔒 lock icon and "CVE-YYYY-NNNNN" summaries). See [CVE Handling Guide](reference/CVE-HANDLING.md) for detailed instructions.

## CVE/Vulnerability Handling

CVE (Common Vulnerabilities and Exposures) tickets require special handling and cannot be listed by the `/list-unscrubbed` command due to API security restrictions.

**For CVE tickets:**
- Manually check the Jira dashboard to find CVE tickets (🔒 icon)
- Follow the [CVE Handling Guide](reference/CVE-HANDLING.md) for triage and remediation
- Use `yarn audit` to scan for vulnerabilities in uhc-portal
- Determine if the CVE affects the frontend (npm) or infrastructure (Go, Python, etc.)
- Post investigation findings to the Jira ticket

## Workflow Phases

### Phase 1: Scrub (`/scrub`) - BUG EVALUATION

**Purpose**: Evaluate a specific bug from the unscrubbed list

**Process**:
- Determine if it's really a bug (vs story/task)
- Check reproducibility
- Identify blockers (needs-uxd, backend dependencies)
- Verify priority level (Blocker → Critical → Major → Normal → Minor)
- Recommend Jira updates (manual for now)
- Add 'scrubbed' label when ready

**Output**: `artifacts/bugfix/scrubbing/scrub-report-{issue-key}.md`

**When to use**: After running `/list-unscrubbed` to pick a specific bug, or if you already have a Jira URL

### Phase 2: Reproduce (`/reproduce`)

**Purpose**: Systematically reproduce the bug in a controlled environment

**Process**:
- Parse bug report and extract key information
- Set up environment (local dev or staging)
- Attempt reproduction with variations
- Document minimal reproduction steps
- Assess severity

**Output**: `artifacts/bugfix/reports/reproduction-{issue-key}.md`

**When to use**: After scrubbing, or start here for already-scrubbed bugs

### Phase 3: Diagnose (`/diagnose`)

**Purpose**: Perform root cause analysis

**Process**:
- Locate relevant code in uhc-portal
- Trace execution flow
- Examine git history
- Form and test hypotheses
- Assess impact and similar patterns
- Recommend fix strategy

**Output**: `artifacts/bugfix/analysis/root-cause-{issue-key}.md`

**When to use**: After successful reproduction

### Phase 4: Fix (`/fix`)

**Purpose**: Implement the bug fix following UHC Portal standards

**Process**:
- Create feature branch (`bugfix/OCMUI-{number}-{description}`)
- Implement minimal code changes
- Follow TypeScript, React, and PatternFly conventions
- Reference `.cursor/rules/*.mdc` files
- Run linters and formatters
- Document implementation choices

**Output**: Modified code files + `artifacts/bugfix/fixes/implementation-{issue-key}.md`

**When to use**: After diagnosis, or jump here if root cause is obvious

### Phase 5: Test (`/test`)

**Purpose**: Verify the fix with comprehensive testing

**Process**:
- Write unit tests (Jest + React Testing Library)
- Run `yarn test-changes` to verify coverage
- Create Playwright e2e tests (when UI changes)
- Follow `.cursor/rules/unit-test-rules.mdc` and `.cursor/rules/playwright-e2e-tests-rules.mdc`
- Run full test suite
- Perform manual verification

**Output**: New test files + `artifacts/bugfix/tests/verification-{issue-key}.md`

**When to use**: After implementing the fix

### Phase 6: Document (`/document`)

**Purpose**: Create PR description and Jira update recommendations

**Process**:
- Create PR description using team template
- Include AI attribution if applicable
- Recommend Jira status transitions
- Prepare issue update comments

**Output**: `artifacts/bugfix/docs/` containing PR description, Jira updates, and issue comments

**When to use**: After testing is complete

### Phase 7: PR (`/pr`)

**Purpose**: Create a draft pull request

**Process**:
- Push branch to personal fork
- Create draft PR targeting main
- Provide post-PR guidance
- Remind about review requirements (2 dev + 1 QE)

**Output**: Draft PR + `artifacts/bugfix/docs/pr-summary-{issue-key}.md`

**When to use**: After documentation is complete

## Getting Started

### Prerequisites

- Access to uhc-portal repository
- Access to Jira Defect Manager Dashboard
- Familiarity with the team's PR process

### Quick Start

**For Interruption Catcher duty:**
```
1. Load this workflow in ACP
2. Run /list-unscrubbed to see all bugs needing scrubbing
3. Run /scrub OCMUI-1234 for the bug you want to evaluate
4. Follow the recommended next steps
```

**For an already-scrubbed bug:**
```
1. Load this workflow in ACP
2. Run /reproduce or /diagnose (depending on clarity)
3. Continue through /fix → /test → /document → /pr
```

**For a Blocker/Critical bug:**
```
1. Load this workflow in ACP
2. Jump to /diagnose or /fix immediately
3. Complete remaining phases quickly
```

## Example Usage

### Scenario 1: Interruption Catcher scrubbing new bugs

```
User: "I'm starting Interruption Catcher duty"

Workflow: Runs /list-unscrubbed
→ Shows 6 unscrubbed bugs in a table
→ User picks OCMUI-1234

User: "/scrub OCMUI-1234"

Workflow: Runs /scrub to evaluate the bug
→ Determines it's a valid bug, reproducible, Major priority
→ Recommends /reproduce to confirm

→ /reproduce confirms the bug
→ /diagnose finds root cause
→ /fix implements the solution
→ /test verifies with unit tests and coverage
→ /document prepares PR description
→ /pr creates draft PR
```

### Scenario 2: Blocker bug needs immediate fix

```
User: "Blocker bug - login page is showing white screen. OCMUI-5678"

Workflow: Jumps to /diagnose (scrubbing skipped for Blocker)
→ Identifies missing error boundary
→ /fix adds error handling
→ /test verifies fix + adds tests
→ /document prepares PR
→ /pr creates draft PR

User manually fast-tracks the PR review process
```

### Scenario 3: Bug with unclear reproduction

```
User: "Bug report is vague: OCMUI-9012"

Workflow: Runs /scrub
→ Identifies missing information
→ Recommends adding Jira comment requesting details
→ User waits for reporter to respond

Later:
→ /reproduce attempts reproduction with new info
→ Continues to /diagnose, /fix, etc.
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
  - Extend `BasePage`
  - Use proper test tags (`@ci`, `@smoke`, `@day1`/`@day2`)

### PR Requirements

- **DRAFT first**: Create as draft initially
- **Jira ticket in title**: `OCMUI-XXXX: Brief description`
- **Team template**: Use `.github/pull_request_template.md`
- **AI attribution**: If substantial AI contribution, note in description and commit
- **Size limits**: Max 1000 lines or 30 files
- **Approvals**: 2 dev + 1 QE required

### Jira Workflow

- **To Do** → **Code Review** (when PR created)
- **Code Review** → **Review** (after 2 dev approvals)
- **Review** → **Done** (after merge + staging verification)

## Priority Levels

Following OCMUI Defect Manager standards:

- **Blocker**: Complete halt in critical functionality (login fails, white screens)
- **Critical**: Significant degradation, no workaround
- **Major**: Important features affected, complex workaround
- **Normal**: Moderate impact, straightforward workaround
- **Minor**: Minimal impact

## Artifacts Generated

All workflow artifacts are organized in `artifacts/bugfix/`:

```
artifacts/bugfix/
├── scrubbing/          # Bug scrubbing reports
│   └── scrub-report-{issue-key}.md
├── reports/            # Reproduction reports
│   └── reproduction-{issue-key}.md
├── analysis/           # Root cause analysis
│   └── root-cause-{issue-key}.md
├── fixes/              # Implementation notes
│   └── implementation-{issue-key}.md
├── tests/              # Test verification
│   └── verification-{issue-key}.md
└── docs/               # Documentation and PR materials
    ├── pr-description-{issue-key}.md
    ├── jira-updates-{issue-key}.md
    ├── issue-update-{issue-key}.md
    └── pr-summary-{issue-key}.md
```

## Key Differences from Generic Bugfix Workflow

This workflow is customized for OCMUI:

1. **Scrubbing phase**: Includes Interruption Catcher duties
2. **Priority system**: Uses OCMUI-specific severity levels
3. **Jira integration**: Provides manual update recommendations
4. **UHC Portal standards**: References `.cursor/rules/*.mdc` files
5. **Testing requirements**: `yarn test-changes` + Playwright conventions
6. **PR process**: 2 dev + 1 QE approval workflow
7. **AI attribution**: Team requirement for tracking AI contributions

## References

- **Unscrubbed Bugs**: Use `/list-unscrubbed` command (executes JQL query directly)
- **CVE Handling**: See [reference/CVE-HANDLING.md](reference/CVE-HANDLING.md) for vulnerability triage and remediation
- **UHC Portal Repo**: https://github.com/RedHatInsights/uhc-portal
- **PR Process**: uhc-portal/docs/pull-request-process.md
- **PR Template**: uhc-portal/.github/pull_request_template.md
- **Coding Rules**: uhc-portal/.cursor/rules/*.mdc

## Troubleshooting

**Problem**: Can't reproduce the bug
**Solution**: Document what you tried, request more info from reporter via Jira comment, or discuss in daily post-scrum

**Problem**: Bug is blocked (needs UXD input)
**Solution**: Set Jira blocked field to true, add blocked reason, add needs-uxd label, don't attempt fix until unblocked

**Problem**: PR is too large (>1000 lines / >30 files)
**Solution**: Consider splitting into smaller PRs, or discuss with team lead for exceptions

**Problem**: Test coverage is low
**Solution**: Add more unit tests focusing on edge cases and error paths

**Problem**: Not sure if it's a bug or feature request
**Solution**: Use /scrub phase - it helps determine if it's truly a bug vs a story/task

## Support

For questions or issues with this workflow:
- Reach out in #ocm-console-team Slack channel
- Reference the Interruption Catcher documentation
- Consult with team leads

---

**Created for**: OCMUI/UHC Portal Team
**Workflow Version**: 1.0.0
**Based on**: Ambient Code Platform Bugfix Workflow Template
