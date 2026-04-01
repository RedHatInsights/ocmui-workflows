---
name: fix
description: Implement bug fix following UHC Portal standards
---

# /fix - Implement Bug Fix

## Purpose

Implement the minimal code changes to fix the bug, following UHC Portal team standards for TypeScript, React, and PatternFly.

## Prerequisites

- Root cause analysis identifying what to fix
- Access to uhc-portal codebase with write permissions
- Understanding of the fix strategy

## Process

### 1. Review Fix Strategy

Load the root cause analysis to understand:
- What code needs to change
- Recommended approach
- Potential impact on other code

### 2. Create Feature Branch

```bash
cd /workspace/repos/uhc-portal
git checkout main
git pull origin main
git checkout -b bugfix/OCMUI-{issue-number}-{brief-description}
```

**Branch naming**: `bugfix/OCMUI-1234-fix-cluster-list-filter`

### 3. Implement the Fix

Follow UHC Portal coding standards:

**TypeScript Standards** (from `.cursor/rules/typescript-rules.mdc`):
- Define prop types using TypeScript interfaces
- Avoid `any` - use `unknown` if needed
- Use existing types from `src/types/` directories
- Define return types for all functions
- Use `import type` for type-only imports

**React Standards** (from `.cursor/rules/react-rules.mdc`):
- Use functional components with hooks
- Keep components small and focused
- Use PascalCase for component files
- Boolean props: prefix with `is`, `has`, `can`, `should`
- Don't mutate props or state - create copies
- Avoid inline functions in JSX
- Use `useMemo` for expensive calculations
- Use `useCallback` for functions passed to children

**General Standards** (from `.cursor/rules/general-rules.mdc`):
- Keep functions small and single-purpose
- Use CamelCase for files unless React (tsx/jsx)
- No abbreviations in variable names
- Match filename with default export name
- Use early return pattern
- Extract reusable logic to custom hooks

**PatternFly UI**:
- Use PatternFly components (no custom CSS)
- Use PatternFly utility classes for spacing/layout
- Import from `@patternfly/react-core`, `@patternfly/react-table`, etc.

### 4. Make Minimal Changes

**Principle**: Fix only what's broken.

❌ **Don't:**
- Refactor surrounding code
- Add features
- Fix unrelated issues
- Change formatting of untouched code
- Add unnecessary comments

✅ **Do:**
- Fix the specific bug
- Add issue reference in comments if logic is non-obvious
- Update types if the fix requires it

### 5. Run Linters

```bash
yarn lint
yarn typecheck
```

Fix any errors before proceeding.

### 6. Test Manually

Run the application locally:
```bash
yarn start
```

- Follow the reproduction steps
- Verify the bug no longer occurs
- Check that the fix doesn't break related functionality
- Test edge cases

### 7. Generate Implementation Notes

Create `artifacts/bugfix/implementation-{issue-key}.md`:

```markdown
# Implementation: {OCMUI-XXXX}

**Branch:** bugfix/OCMUI-{number}-{description}

## Changes
- `{file}:{lines}` - {what and why}

## Key Decisions
{If applicable: - {Decision}: {rationale}}

## PR Size
{lines} lines, {files} files - {within/over} limits

{If AI assisted: Assisted by: Claude Sonnet 4.5}

## Next Step
**Issue:** OCMUI-{XXXX}
**Ready for:** /test OCMUI-{XXXX}
```

### 8. Re-read Controller and Return

After generating implementation notes, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- Modified code files in uhc-portal
- `artifacts/bugfix/implementation-{issue-key}.md` — Implementation notes
- Console output with next step: `/test {issue-key}`

## Notes

- **Minimal changes**: Resist the urge to refactor. Fix the bug and nothing else.
- **Standards matter**: Following team conventions makes code review smoother
- **Test manually**: Don't rely only on automated tests - verify with your eyes
- **PR size limits**: If over 1000 lines or 30 files, consider splitting the fix
- **AI attribution**: If AI assisted significantly, note it for the PR description
