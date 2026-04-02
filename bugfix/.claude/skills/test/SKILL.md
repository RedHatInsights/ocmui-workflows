---
name: test
description: Write unit tests, run coverage checks, and create e2e tests
---

# /test - Testing and Verification

## Purpose

Write comprehensive unit tests following UHC Portal standards, verify test coverage with `yarn test-changes`, and create Playwright e2e tests when UI changes are involved.

## Prerequisites

- Bug fix implemented and linters passing
- Access to uhc-portal codebase
- Understanding of what code was changed

## Process

### 1. Review Implementation

Load implementation notes to understand:
- Which files were modified
- What the fix does
- What behavior changed

### 2. Write Unit Tests

Follow UHC Portal unit testing standards (from `.cursor/rules/unit-test-rules.mdc`):

**Testing Principles:**
- Follow Arrange-Act-Assert pattern
- Test behavior, not implementation details
- Write descriptive test names
- Keep mocks simple and focused

**React Testing Library:**
- Import `render`, `screen` from `~/testUtils`
- Use `user` from render for interactions (not `fireEvent`)
- Query priority: `getByRole` > `getByLabelText` > `getByPlaceholderText` > `getByText` > `getByTestId`
- Use `waitFor` for async operations
- Use `checkAccessibility` utility when appropriate

**Best Practices:**
- Avoid mocking child React components unless necessary
- Use `jest.clearAllMocks()` in `beforeEach` or `afterEach`
- Test error states and loading states
- Use `jest.spyOn` for mocking specific methods
- Use `jest.mock` at module level for consistency
- Use descriptive assertion messages when failures might be unclear

**Example Test Structure:**
```typescript
import { render, screen } from '~/testUtils';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should handle cluster selection correctly', async () => {
    // Arrange
    const mockOnSelect = jest.fn();
    const { user } = render(<MyComponent onSelect={mockOnSelect} />);

    // Act
    await user.click(screen.getByRole('button', { name: /select cluster/i }));

    // Assert
    expect(mockOnSelect).toHaveBeenCalledWith(expect.objectContaining({
      id: 'cluster-123'
    }));
  });

  it('should display error message when API fails', async () => {
    // Arrange
    jest.spyOn(console, 'error').mockImplementation();
    const { user } = render(<MyComponent />);

    // Act
    await user.click(screen.getByRole('button', { name: /submit/i }));

    // Assert
    await waitFor(() => {
      expect(screen.getByText(/error occurred/i)).toBeInTheDocument();
    });
  });
});
```


### 2.5. Follow Test File Standards from ESLint Config

**IMPORTANT: Before writing test files, understand the linting rules.**

**Required import order (from [.eslintrc](https://github.com/RedHatInsights/uhc-portal/blob/main/.eslintrc) lines 103-123):**

```typescript
// Group 1: react first, then a-z packages
import React from 'react';
import { Formik } from 'formik';

// Group 2: @ packages
import { Alert } from '@patternfly/react-core';

// Group 3: ~ internal imports (sorted alphabetically)
import shouldShowModal from '~/components/common/Modal/ModalSelectors';
import { useEditCluster } from '~/queries/ClusterDetailsQueries/useEditCluster';
import { render, screen } from '~/testUtils';

// Group 4: Parent directory (..)
import { helper } from '../utils/helper';

// Group 5: Current directory (.)
import EditClusterWideProxyDialog from './EditClusterWideProxyDialog';
import EditClusterWideProxyForm from './EditClusterWideProxyForm';

// Group 6: THEN jest.mock statements
jest.mock('~/queries/ClusterDetailsQueries/useEditCluster');
jest.mock('~/components/common/Modal/ModalSelectors');
```

**Critical rules:**
- ❌ NO imports after `jest.mock()` - causes import/first error
- ❌ NO snapshot tests - forbidden by `.eslintrc` line 102
- ✅ Use `user` from render (not `fireEvent`) - required by `.eslintrc` line 98
- ✅ Complete mock return types - check source file for exact properties

**Verify mock return types against source:**
```bash
# Before writing mocks, check the actual hook
grep -A 15 "return {" /workspace/repos/uhc-portal/src/queries/path/to/hook.ts
```

Example - complete mock for `useMutation`:
```typescript
// ✅ CORRECT - includes all properties
const mockReturn = {
  data: undefined,       // Required
  isPending: false,
  isError: false,
  error: null,
  isSuccess: false,      // Required
  mutate: jest.fn(),
  reset: jest.fn(),
};

// ❌ WRONG - missing data, isSuccess
const mockReturn = {
  isPending: false,
  isError: false,
  error: null,
  mutate: jest.fn(),
  reset: jest.fn(),
};
```

### 3. Create Regression Test

Write a test that:
- **Fails** without your fix (verifies the bug existed)
- **Passes** with your fix (proves it's fixed)

This ensures the bug doesn't come back.

### 4. Run Test Suite

**Run modified file tests:**
```bash
yarn test {test-file-path}
```

**Run full test suite:**
```bash
yarn test
```

Ensure all tests pass.

### 4.5. Verify Test File Linting (MANDATORY)

**After creating test files, validate them against ESLint rules:**

```bash
cd /workspace/repos/uhc-portal

# Lint specific test file
yarn lint src/path/to/test.tsx

# Auto-fix import order and formatting
yarn lint --fix src/path/to/test.tsx

# TypeScript validation
yarn typecheck
```

**Common test file errors:**

1. **Import order** - Automatically fixable with `yarn lint --fix`
2. **Unused variables** - Remove unused destructured values
3. **Incomplete mock types** - Add missing properties from source
4. **Wrong testing library methods** - Use `user.click()` not `fireEvent.click()`

**Checklist before proceeding:**
- [ ] All imports before `jest.mock()`
- [ ] Import order: react → @packages → ~/internal → ../ → ./
- [ ] Complete mock return types (matches source exactly)
- [ ] No unused variables
- [ ] Using `user` from render (not `fireEvent`)
- [ ] No snapshot tests

**DO NOT PROCEED to coverage check if linting fails.**


### 5. Check Test Coverage

**Critical step for UHC Portal:**
```bash
yarn test-changes
```

This shows coverage for only the changed code.

**Coverage expectations:**
- Aim for high coverage on modified code
- It's okay if not 100% (some code is hard to test)
- Focus on testing the bug fix and edge cases

If coverage is low, add more tests.

### 5.5. Build Verification (MANDATORY)

**After tests pass, verify the code builds successfully:**

```bash
cd /workspace/repos/uhc-portal
yarn build
```

**Why this matters:**
- `yarn typecheck` validates types but doesn't check webpack compilation
- `yarn build` catches import errors, module resolution issues, and runtime type errors
- Build failures in CI are more expensive than catching them locally

**If build fails:**
1. Check error message for specific file and line
2. Common issues:
   - Circular dependencies
   - Missing type exports
   - Invalid import paths
   - TypeScript errors not caught by typecheck

**DO NOT proceed to documentation if build fails.**


### 6. Create E2E Tests (if UI changes)

**When e2e tests are needed:**
- New UI components added
- UI interaction flow changed
- Critical user paths affected

**Playwright e2e standards** (from `.cursor/rules/playwright-e2e-tests-rules.mdc`):

**Page Objects:**
- Every page object extends `BasePage`
- File naming: `{feature}-page.ts`, class: `{Feature}Page`
- Locator methods are synchronous (return `Locator`)
- Action methods are async (return `Promise<void>`)
- Every page has `is{PageName}()` method for validation
- Selector priority: `getByRole` > `getByLabel` > `getByText` > `getByTestId`
- Never use CSS selectors or dynamic IDs
- Chain locators for scoped queries

**Test Specs:**
- Import `test`, `expect` from custom fixtures (`../../fixtures/pages`)
- Use `test.describe.serial` for multi-step flows with shared state
- Use regular `test.describe` for independent tests
- Every describe includes tags: `{ tag: ['@smoke', '@ci', '@rosa'] }`
- Use `navigateTo` fixture for navigation
- Call `is{PageName}()` in `test.beforeAll` to validate
- Tag `@ci` for fast, side-effect-free tests
- Tag `@smoke` for critical Day 0/Day 1 paths
- Tag `@day1` or `@day2` to indicate lifecycle phase
- Never use `page.waitForTimeout()` or `page.waitForLoadState('networkidle')`

**Example e2e test:**
```typescript
import { test, expect } from '../../fixtures/pages';

test.describe.serial('Cluster list filtering', { tag: ['@ci', '@day1', '@rosa'] }, () => {
  test.beforeAll(async ({ navigateTo, clusterListPage }) => {
    await navigateTo('/clusters');
    await clusterListPage.isClusterListPage();
  });

  test('should filter clusters by name', async ({ clusterListPage }) => {
    await clusterListPage.filterByName('test-cluster');
    expect(await clusterListPage.getClusterCount()).toBe(1);
  });
});
```

### 7. Generate Test Report

Create `artifacts/bugfix/tests/verification-{issue-key}.md`:

````markdown
# Test Verification Report: {Issue Key}

## Bug Summary

- **Issue**: {OCMUI-XXXX}
- **Files tested**: {list test files}

## Unit Tests

### New Tests Added

**{test-file-1}.test.tsx**
- `should {description}` — {what this tests}
- `should {description}` — {what this tests}

**{test-file-2}.test.tsx**
- `should {description}` — {what this tests}

### Regression Test

✅ **Test that proves bug is fixed:**
- Test: `{test name}`
- File: `{test-file}.test.tsx`
- **Without fix**: ❌ Fails (verifies bug existed)
- **With fix**: ✅ Passes (proves it's fixed)

### Test Results

```
yarn test
{paste relevant output}

PASS  src/components/ClusterList.test.tsx
PASS  src/hooks/useClusterFilter.test.tsx

Test Suites: 2 passed, 2 total
Tests:       8 passed, 8 total
```

## Coverage Check

```
yarn test-changes
{paste output}

File                       | Stmts | Branch | Funcs | Lines
---------------------------|-------|--------|-------|-------
src/components/ClusterList.tsx | 95.2  | 87.5   | 100   | 94.8
src/hooks/useClusterFilter.ts  | 88.9  | 75.0   | 100   | 88.9
```

**Coverage Assessment:**
- Modified code coverage: {percentage}%
- Edge cases tested: {yes/no}
- Error paths tested: {yes/no}

{If low coverage, explain why or note that more tests are needed}

## E2E Tests

{If applicable:}

**Tests added:**
- `{spec-file}.spec.ts` — {what it tests}

**Test execution:**
```
yarn test:e2e
{relevant output}
```

{If not applicable: E2E tests not required for this change}

## Manual Verification

✅ **Manual test checklist:**
- [ ] Original bug reproduction steps no longer trigger the bug
- [ ] Related functionality still works
- [ ] Edge cases tested (empty states, errors, etc.)
- [ ] No new console errors
- [ ] No new TypeScript errors
- [ ] Accessibility checked (keyboard navigation, screen readers)

## Test Quality

**Standards followed:**
- [ ] Arrange-Act-Assert pattern used
- [ ] Descriptive test names
- [ ] Testing behavior, not implementation
- [ ] Mocks are simple and focused
- [ ] React Testing Library best practices followed
- [ ] Accessibility testing included (if UI changes)

## Issues Found During Testing

{If tests revealed issues:}
- {Issue 1} — {how you addressed it}
- {Issue 2} — {how you addressed it}

{If no issues: No issues found}

## Confidence Level

**High** / **Medium** / **Low**

{Explain your confidence in the fix based on test results}

## Next Steps

Ready to proceed to `/document` phase to prepare PR description.
````

### 8. Re-read Controller and Return

After generating the test report, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- New/modified test files in uhc-portal
- `artifacts/bugfix/tests/verification-{issue-key}.md` — Test verification report

## Success Criteria

After running this phase:
- [ ] Regression test created (fails without fix, passes with fix)
- [ ] Unit tests written following team standards
- [ ] Full test suite passes
- [ ] `yarn test-changes` run to verify coverage
- [ ] E2E tests created if UI changes (following Playwright standards)
- [ ] Manual verification complete
- [ ] Test quality documented

## Notes

- **Regression test is critical**: This proves the bug is fixed and prevents it from coming back
- **yarn test-changes is required**: This is a UHC Portal standard for verifying coverage on modified code
- **E2E tests when needed**: If the bug affected UI interactions, add Playwright tests following the team's page object patterns
- **Don't skip manual testing**: Automated tests don't catch everything
- **Coverage isn't everything**: High quality tests > high coverage of low quality tests
