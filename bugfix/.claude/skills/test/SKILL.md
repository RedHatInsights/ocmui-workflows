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

### 3. Create Regression Test

Write a test that:
- **Fails** without your fix (verifies the bug existed)
- **Passes** with your fix (proves it's fixed)

This ensures the bug doesn't come back.

### 4. Run Test Suite

```bash
yarn test {test-file-path}  # Run modified file tests
yarn test                    # Run full test suite
```

Ensure all tests pass.

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
- Regression test: `{test name}` - proves bug is fixed

## Coverage
`yarn test-changes` - {X}% on modified code

## E2E Tests
{If applicable: test file and description}
{If not: "Not required for this change"}

## Status
{All pass / Failures found and fixed / Pending execution}

{If issues found during testing: Brief note}

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
- **E2E tests when needed**: If the bug affected UI interactions, add Playwright tests
- **Don't skip manual testing**: Automated tests don't catch everything
