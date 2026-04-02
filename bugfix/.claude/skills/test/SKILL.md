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

Follow UHC Portal unit testing standards (from `.cursor/rules/unit-test-rules.mdc` and `docs/unit-testing.md`):

**Testing Principles:**
- Follow **Arrange-Act-Assert** pattern
- Test behavior, not implementation details
- Write descriptive test names: "expected result when conditions"
- Keep mocks simple and focused

**Use `it()` not `test()`:**
```javascript
// ✅ Correct
it('displays "hello" when "hello" is sent as a prop', () => {});

// ❌ Wrong
test('displays hello', () => {});
```

**React Testing Library Best Practices:**
```javascript
import { render, screen, waitFor } from '~/testUtils';

it('shows welcome message when user clicks button', async () => {
  // Arrange
  const { user } = render(<MyComponent />);
  expect(screen.queryByText('Welcome')).not.toBeInTheDocument();

  // Act
  await user.click(screen.getByRole('button', { name: /click me/i }));

  // Assert
  expect(screen.getByText('Welcome')).toBeInTheDocument();
});
```

**Query Priority (in order):**
1. `getByRole` - accessibility tree role (best)
2. `getByLabelText` - form elements
3. `getByPlaceholderText` - form elements
4. `getByText` - non-interactive elements
5. `getByDisplayValue` - form elements
6. `getByAltText` - images
7. `getByTitle` - only if title attribute is appropriate
8. `getByTestId` - last resort (explain why others don't work)

**User Interactions:**
- Use `user` from render (NOT `fireEvent` or `userEvent` directly)
- Clear inputs before typing: `await user.clear(input); await user.type(input, 'text')`
- Wait for elements to be enabled before clicking

**Assertions:**
- Use `toBeInTheDocument()` for existence checks (not `toBeTruthy()`)
- Use `toBeVisible()` for CSS visibility changes
- Use descriptive assertion messages when failure might be unclear
- Avoid try/catch blocks in tests

**Mocking:**
- Import from `~/testUtils` not `@testing-library/react`
- Use `jest.clearAllMocks()` in `beforeEach` or `afterEach`
- Mock external dependencies and API calls consistently
- Use `jest.spyOn` for mocking specific methods
- Use `jest.mock` at module level for consistency
- Avoid mocking child React components unless necessary

**Redux Testing:**
- Use `withState` utility for Redux testing
- Test components with Redux state properly

**Custom Hooks:**
- Test with `renderHook` when isolated testing is needed

**Test Fixtures:**
- Create meaningful test fixtures
- Move fixtures to separate files for reusability

**Test Independence:**
- Each test must run alone successfully
- Don't rely on test execution order
- Clear state between tests

**Accessibility Testing:**
```javascript
import { checkAccessibility } from '~/testUtils';

it('is accessible on initial render', async () => {
  const { container } = render(<MyComponent />);
  await checkAccessibility(container);
});
```

**Async Operations:**
- Use `waitFor` for async operations and state updates
- Use `findBy` queries instead of `waitFor` when possible
- Use `waitForElementToBeRemoved` for disappearance

**Avoid Snapshot Tests:**
- Don't use `toMatchSnapshot()` - they don't catch bugs
- Instead, test specific behavior

**Test Structure:**
```javascript
describe('<MyComponent />', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('Feature A', () => {
    it('does X when Y happens', () => {
      // Arrange
      // Act
      // Assert
    });
  });
});
```

**TDD Approach (Recommended):**
```javascript
it.todo('displays error message when API call fails');
it.todo('shows loading spinner while fetching data');
```
Write `it.todo()` tests before implementing, then fill them in.

**Skip Broken Tests (with explanation):**
```javascript
it.skip('is accessible after modal opens', async () => {
  // NOTE: Skipped due to PatternFly accessibility issue
  // Tracking: https://github.com/patternfly/patternfly-react/issues/12345
  const { container } = render(<MyComponent />);
  await checkAccessibility(container);
});
```


### 2.5. Review Test File Linting Rules

**Before writing test files, review critical linting rules:**

**Import order from [.eslintrc](https://github.com/RedHatInsights/uhc-portal/blob/main/.eslintrc) (lines 103-123):**
```typescript
// 1. React and external packages
import React from 'react';
import { Formik } from 'formik';

// 2. @ packages
import { Alert } from '@patternfly/react-core';

// 3. ~ internal imports (sorted alphabetically)
import { useEditCluster } from '~/queries/ClusterDetailsQueries/useEditCluster';
import { render, screen } from '~/testUtils';

// 4. Relative imports (.., .)
import EditClusterWideProxyDialog from './EditClusterWideProxyDialog';

// 5. THEN jest.mock() statements
jest.mock('~/queries/ClusterDetailsQueries/useEditCluster');
```

**Critical rules to follow:**
- ✅ All imports BEFORE `jest.mock()` - violating causes `import/first` error
- ✅ Complete mock return types - check source file:
  ```bash
  grep -A 10 "return {" src/queries/path/to/hook.ts
  ```
- ✅ No unused variables (causes `TS6133` errors)
- ✅ Use `user` from render, NOT `fireEvent` (`.eslintrc` line 98)
- ❌ NO snapshot tests (forbidden by `.eslintrc` line 102)

**Reference:** [ESLint & TypeScript Guide](../reference/eslint-typescript-guide.md) for complete test file template and examples.

### 3. Create Regression Test

Write a test that:
- **Fails** without your fix (verifies the bug existed)
- **Passes** with your fix (proves it's fixed)

This ensures the bug doesn't come back.

Example:
```javascript
describe('Regression: OCMUI-4183 customer scenario', () => {
  it('should not send proxy object when adding CA cert to cluster with existing proxy', async () => {
    // This is the exact scenario from the bug report
    const cluster = createMockCluster({ proxy: existingProxy });
    const { user } = render(<EditProxyDialog cluster={cluster} />);

    // Customer wants to add CA cert without touching proxy settings
    await user.type(screen.getByTestId('trust-bundle'), certData);

    // Wait for button to be enabled
    const saveButton = screen.getByRole('button', { name: /save/i });
    await waitFor(() => expect(saveButton).toBeEnabled());
    await user.click(saveButton);

    await waitFor(() => {
      expect(mockMutate).toHaveBeenCalledTimes(1);
    });

    // The fix: Should NOT include proxy object
    expect(mockMutate.mock.calls[0][0].cluster.proxy).toBeUndefined();
  });
});
```

### 4. Run Test Suite

```bash
yarn test src/path/to/file.test.tsx  # Run specific test file
yarn test                             # Run all tests
yarn test-local                       # Reduced CPU usage for local dev
```

Ensure all tests pass.

### 5. Check Test Coverage

**Critical step for UHC Portal:**
```bash
yarn test-changes
```

This shows coverage for **only the changed code**.

**Coverage expectations:**
- Aim for high coverage on modified code
- It's okay if not 100% (some code is hard to test)
- Focus on testing the bug fix and edge cases
- Test error states and loading states

If coverage is low, add more tests.

### 6. Create E2E Tests (if UI changes)

**When e2e tests are needed:**
- New UI components added
- UI interaction flow changed
- Critical user paths affected

Follow Playwright standards from `.cursor/rules/playwright-e2e-tests-rules.mdc`.

### 7. Generate Test Report

Create `artifacts/bugfix/tests-{issue-key}.md`:

```markdown
# Tests: {OCMUI-XXXX}

## Tests Added
**{test-file}.test.tsx** - {#} test cases
- `{test name}` - {description}
- Regression test: `{test name}` - proves bug is fixed

## Test Standards Applied
- ✅ Used `it()` not `test()`
- ✅ AAA pattern (Arrange-Act-Assert)
- ✅ Descriptive test names: "expected result when conditions"
- ✅ Used `user` from render for interactions
- ✅ Waited for elements to be enabled before clicking
- ✅ Used `toBeInTheDocument()` for existence checks
- ✅ Query priority: getByRole first
- ✅ Accessibility tested with `checkAccessibility()`
- ✅ Regression test included

## Coverage
`yarn test-changes` - {X}% on modified code

{If coverage < 80%: Explanation of why or plan to improve}

## E2E Tests
{If applicable: test file and description}
{If not: "Not required for this change"}

## Test Execution
```bash
yarn test {file-path}
```
All tests passing ✅

## Next Step
**Issue:** OCMUI-{XXXX}
**Ready for:** /draft-pr OCMUI-{XXXX}
```

### 8. Re-read Controller and Return

After generating the test report, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- New/modified test files in uhc-portal
- `artifacts/bugfix/tests-{issue-key}.md` — Test verification report
- Console output with next step: `/draft-pr {issue-key}`

## Notes

- **Regression test is critical**: This proves the bug is fixed and prevents it from coming back
- **yarn test-changes is required**: This is a UHC Portal standard for verifying coverage on modified code
- **Test naming matters**: Use format "expected result when conditions"
- **Use it() not test()**: Team standard
- **Wait for elements**: Buttons must be enabled, data must load
- **Test independence**: Each test must run alone
- **E2E tests when needed**: If the bug affected UI interactions, add Playwright tests
- **Don't skip manual testing**: Automated tests don't catch everything
- **Follow .cursor/rules**: Reference `unit-test-rules.mdc` for all standards
