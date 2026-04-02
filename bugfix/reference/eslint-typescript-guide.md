# ESLint & TypeScript Configuration Reference

Quick reference for uhc-portal linting rules to prevent common errors during bugfix workflow.

## Source Configurations

- **ESLint:** [.eslintrc](https://github.com/RedHatInsights/uhc-portal/blob/main/.eslintrc)
- **TypeScript:** [tsconfig.json](https://github.com/RedHatInsights/uhc-portal/blob/main/tsconfig.json)
- **TypeScript Lint:** [tsconfig.lint.json](https://github.com/RedHatInsights/uhc-portal/blob/main/tsconfig.lint.json)

---

## Critical Import Order Rules

**From `.eslintrc` lines 103-123 (simple-import-sort/imports)**

**Required order:**
```typescript
// Group 1: react first, next second, then packages (a-z)
import React from 'react';
import { Formik } from 'formik';

// Group 2: Packages starting with @
import { Alert, Button } from '@patternfly/react-core';

// Group 3: Packages starting with ~ (internal path alias)
import shouldShowModal from '~/components/common/Modal/ModalSelectors';
import { useEditCluster } from '~/queries/ClusterDetailsQueries/useEditCluster';
import { render, screen } from '~/testUtils';

// Group 4: Parent directory imports (../)
import { helper } from '../utils/helper';

// Group 5: Current directory imports (./)
import EditClusterWideProxyDialog from './EditClusterWideProxyDialog';

// Group 6: Style imports
import './styles.css';
```

**Common violation:**
```typescript
// ❌ WRONG - imports after jest.mock
import React from 'react';

jest.mock('~/queries/useHook');

import { useHook } from '~/queries/useHook';  // ← import/first error
```

**Fix:**
```typescript
// ✅ CORRECT - all imports first
import React from 'react';

import { useHook } from '~/queries/useHook';

jest.mock('~/queries/useHook');
```

---

## Testing Library Rules

**From `.eslintrc` lines 92-101**

### Required:
- ✅ `testing-library/prefer-user-event` - Use `user` from render, not `fireEvent`
- ✅ `testing-library/prefer-find-by` - Use `findBy` instead of `waitFor` + `getBy`
- ❌ `no-snapshot-testing/no-snapshot-testing` - NO snapshot tests allowed

### Examples:

```typescript
// ✅ CORRECT - use user from render
const { user } = render(<Component />);
await user.click(screen.getByRole('button'));

// ❌ WRONG - don't use fireEvent
import { fireEvent } from '@testing-library/react';
fireEvent.click(button);  // Violates testing-library/prefer-user-event
```

```typescript
// ✅ CORRECT - use findBy
const element = await screen.findByText('Loading complete');

// ❌ WRONG - don't use waitFor + getBy
await waitFor(() => {
  expect(screen.getByText('Loading complete')).toBeInTheDocument();
});
```

```typescript
// ❌ WRONG - no snapshots
expect(container).toMatchSnapshot();  // Violates no-snapshot-testing
```

---

## Import Restrictions

**From `.eslintrc` lines 22-53**

### PatternFly Icons
```typescript
// ✅ CORRECT - full ESM path
import CheckIcon from '@patternfly/react-icons/dist/esm/icons/check-icon';

// ❌ WRONG - barrel import
import { CheckIcon } from '@patternfly/react-icons';
```

### API Request Service
```typescript
// ✅ CORRECT - use path alias
import apiRequest from '~/services/apiRequest';

// ❌ WRONG - relative import
import apiRequest from '../../../services/apiRequest';
```

---

## React Hooks

**From `.eslintrc` line 61**

```json
"react-hooks/exhaustive-deps": "error"
```

**Rule:** All dependencies in `useEffect`, `useCallback`, `useMemo` must be listed.

```typescript
// ❌ WRONG - missing dependency
useEffect(() => {
  fetchData(userId);
}, []);  // Missing userId in deps

// ✅ CORRECT
useEffect(() => {
  fetchData(userId);
}, [userId]);

// Or disable if intentional (with comment explaining why)
useEffect(() => {
  fetchData(userId);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);  // Only run on mount
```

---

## TypeScript Configuration

### From tsconfig.json

**Key compiler options:**
```json
{
  "strict": true,                    // All strict checks enabled
  "noUnusedLocals": true,           // Error on unused variables
  "noUnusedParameters": true,       // Error on unused function params
  "esModuleInterop": true,
  "allowSyntheticDefaultImports": true,
  "paths": {
    "~/*": ["src/*"]                // Path alias for imports
  }
}
```

### Common TypeScript Errors

#### 1. Unused Variables
```typescript
// ❌ TS6133: 'rerender' is declared but its value is never read
const { rerender } = render(<Component />);
// Solution: Remove it or use it

// ✅ CORRECT - only destructure what you use
render(<Component />);
```

#### 2. Incomplete Type Definitions
```typescript
// ❌ Type error - missing properties
const mockReturn = {
  isPending: false,
  mutate: jest.fn(),
};

// ✅ CORRECT - complete type matching source
const mockReturn = {
  data: undefined,        // Required
  isPending: false,
  isError: false,
  error: null,
  isSuccess: false,       // Required
  mutate: jest.fn(),
  reset: jest.fn(),
};
```

---

## Test File Template

Based on all the above rules, here's the correct test file structure:

```typescript
// 1. React and external imports (Group 1)
import React from 'react';

// 2. @ packages (Group 2)
import { render as rtlRender } from '@testing-library/react';

// 3. ~ internal imports (Group 3) - sorted alphabetically
import shouldShowModal from '~/components/common/Modal/ModalSelectors';
import { useEditCluster } from '~/queries/ClusterDetailsQueries/useEditCluster';
import { render, screen } from '~/testUtils';

// 4. Relative imports - parent first (Group 4)
import { helper } from '../utils/helper';

// 5. Relative imports - current dir (Group 5)
import EditClusterWideProxyDialog from './EditClusterWideProxyDialog';

// 6. Mock setup (AFTER all imports)
jest.mock('~/queries/ClusterDetailsQueries/useEditCluster');
jest.mock('~/components/common/Modal/ModalSelectors');

// 7. Type mocked functions
const mockUseEditCluster = useEditCluster as jest.MockedFunction<typeof useEditCluster>;
const mockShouldShowModal = shouldShowModal as jest.MockedFunction<typeof shouldShowModal>;

describe('<EditClusterWideProxyDialog />', () => {
  // 8. Complete mock return type
  const defaultMockReturn = {
    data: undefined,
    isPending: false,
    isError: false,
    error: null,
    isSuccess: false,
    mutate: jest.fn(),
    reset: jest.fn(),
  };

  beforeEach(() => {
    jest.clearAllMocks();
    mockUseEditCluster.mockReturnValue(defaultMockReturn);
  });

  it('renders when modal is open', async () => {
    mockShouldShowModal.mockReturnValue(true);

    // 9. Use user from render (not fireEvent)
    const { user } = render(<EditClusterWideProxyDialog />);

    // 10. Use findBy for async (not waitFor + getBy)
    const element = await screen.findByRole('button');

    // 11. Use proper assertions
    expect(element).toBeInTheDocument();
  });
});
```

---

## Quick Checklist for Test Files

Before committing test files, verify:

- [ ] All `import` statements before `jest.mock()`
- [ ] Import order: react → @packages → ~/internal → ../ → ./
- [ ] No `fireEvent` - use `user` from render
- [ ] No `waitFor` + `getBy` - use `findBy`
- [ ] No snapshot tests
- [ ] Complete mock return types (check source file!)
- [ ] No unused variables (TypeScript will error)
- [ ] PatternFly icons use full ESM paths
- [ ] Internal imports use `~/` not relative paths

---

## Auto-Fix Commands

```bash
# Fix import order and formatting
yarn lint --fix

# Check specific file
yarn lint src/components/path/to/file.tsx

# TypeScript errors (no auto-fix)
yarn typecheck

# Full verification
yarn lint && yarn typecheck
```

---

## Workflow Integration

### For /fix phase - After implementing code:

```bash
# Verify code follows linting rules
cd /workspace/repos/uhc-portal
yarn lint
yarn typecheck
```

### For /test phase - After creating test files:

```bash
# Verify test file follows all rules
yarn lint src/path/to/test.tsx

# Auto-fix what's possible
yarn lint --fix

# Check types
yarn typecheck
```

---

*Reference guide created for OCMUI bugfix workflow - based on uhc-portal configs*
