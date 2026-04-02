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
- Similar patterns to address

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
- Address similar patterns if identified in diagnosis
- Add issue reference in comments if logic is non-obvious
- Update types if the fix requires it

### 5. Handle Similar Patterns

If diagnosis identified similar code:

- Fix them in the same PR if they're clearly the same bug
- Skip if uncertain - create separate issues
- Document decision in implementation notes

### 6. Run Linters (MANDATORY)

**MUST PASS before proceeding:**
```bash
cd /workspace/repos/uhc-portal
yarn install --frozen-lockfile  # if needed
yarn lint && yarn typecheck
```

**If errors, try auto-fix:**
```bash
yarn lint --fix  # Fixes import order, formatting
```

**Common issues:**
- Import order: All imports before `jest.mock()`
- Unused variables: Remove or prefix with `_`

**Reference:** [ESLint Guide](../reference/eslint-typescript-guide.md) | [.eslintrc](https://github.com/RedHatInsights/uhc-portal/blob/main/.eslintrc) | [tsconfig.json](https://github.com/RedHatInsights/uhc-portal/blob/main/tsconfig.json)

**DO NOT PROCEED if linters fail.** Fix all errors before continuing.

### 7. Test Manually

Run the application locally:
```bash
yarn start
```

- Follow the reproduction steps
- Verify the bug no longer occurs
- Check that the fix doesn't break related functionality
- Test edge cases

### 8. Generate Implementation Notes

Create `artifacts/bugfix/fixes/implementation-{issue-key}.md`:

````markdown
# Implementation Notes: {Issue Key}

## Bug Summary

- **Issue**: {OCMUI-XXXX}
- **Branch**: `bugfix/OCMUI-{number}-{description}`

## Root Cause (Summary)

{Brief recap of the root cause}

## Changes Made

### Files Modified

- `{file}:{lines}` — {what changed and why}
- `{file}:{lines}` — {what changed and why}

### Code Changes

**{File 1}**
```typescript
// Before
{show the problematic code}

// After
{show the fixed code}

// Why: {explain the fix}
```

**{File 2}**
{repeat for each file}

## Standards Applied

✅ TypeScript:
- [ ] Proper interfaces defined
- [ ] No `any` types used
- [ ] Return types specified
- [ ] Type-only imports used

✅ React:
- [ ] Functional component patterns
- [ ] Proper hook usage
- [ ] No inline functions in JSX
- [ ] Props not mutated

✅ PatternFly:
- [ ] PatternFly components used
- [ ] No custom CSS added
- [ ] Utility classes for layout

## Similar Patterns Addressed

{If you fixed related patterns:}
- Also fixed in `{file}:{line}` — {reason}
- Did NOT fix in `{file}:{line}` — {reason why}

## Manual Verification

✅ Verified:
- [ ] Bug no longer reproduces
- [ ] Original functionality still works
- [ ] Edge cases tested
- [ ] No console errors
- [ ] Linters pass (`yarn lint && yarn typecheck`)

## Implementation Decisions

**{Decision 1}**: {what you chose and why}
**{Decision 2}**: {what you chose and why}

## PR Size Check

- **Lines changed**: {estimate if known}
- **Files changed**: {count}
- **Within limits**: {yes/no} (max 1000 lines / 30 files)

{If over limits, note: "PR is large - consider breaking into smaller changes"}

## AI Attribution

{If substantial AI assistance was used:}
**AI Contribution**: This fix was assisted by {tool name}

## Next Steps

Ready to proceed to `/test` phase for unit tests and coverage verification.
````

### 9. Re-read Controller and Return

After generating implementation notes, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- Modified code files in uhc-portal
- `artifacts/bugfix/fixes/implementation-{issue-key}.md` — Implementation notes

## Success Criteria

After running this phase:
- [ ] Feature branch created
- [ ] Code changes implement the fix
- [ ] UHC Portal standards followed
- [ ] Linters pass (`yarn lint && yarn typecheck`)
- [ ] Manual verification complete
- [ ] Implementation documented
- [ ] PR size within limits (or noted if over)

## Notes

- **Minimal changes**: Resist the urge to refactor. Fix the bug and nothing else.
- **Standards matter**: Following team conventions makes code review smoother
- **Test manually**: Don't rely only on automated tests - verify with your eyes
- **PR size limits**: If over 1000 lines or 30 files, consider splitting the fix
- **AI attribution**: If AI assisted significantly, note it for the PR description
- **Mandatory linting**: Lint errors will fail CI - catch them early
