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

Follow UHC Portal coding standards from `.cursor/rules/`:

**General Standards** (`.cursor/rules/general-rules.mdc`):
- Keep functions small and single-purpose
- Use CamelCase for files unless React (tsx/jsx)
- No abbreviations in variable names
- Match filename with default export name
- Use descriptive names for variables and functions
- Use `UPPER_SNAKE_CASE` for true constants
- Use early return pattern
- Handle loading and error states properly with user-friendly messages
- Use ternary operators instead of if/else when possible (but don't nest)
- Extract reusable logic to custom hooks or functions

**React Standards** (`.cursor/rules/react-rules.mdc`):
- Use functional components with hooks (not class components)
- Keep components small and focused on single responsibility
- Use PascalCase for component files
- Boolean props: prefix with `is`, `has`, `can`, `should`
- Use camelCase for prop names
- Don't mutate props or state - create copies
- Don't store derived data in state - compute it
- Avoid index as key prop - use unique IDs
- Avoid inline functions in JSX
- Use ternary pattern: `{show ? <Comp /> : null}` instead of `{show && <Comp />}`
- Use `useMemo` for expensive calculations
- Use `useCallback` for functions passed to children
- Implement proper cleanup in useEffect hooks
- Use Redux hooks (`useSelector`, `useDispatch`) not connect HOC
- Use react-query for server state management
- Avoid custom CSS - use PatternFly variables or utility classes

**React Query Standards** (`.cursor/rules/react-query-rules.mdc`):
*When fixing bugs involving API calls or data fetching:*
- Create custom hooks that wrap React Query hooks
- Add data sent to API as part of the query key
- Use `enabled` option to prevent queries when dependencies unavailable
- Use `formatErrorData` helper for consistent error handling
- Implement proper loading states (`isLoading`, `isPending`, `isFetching`)
- Use `useQuery` for fetching, `useMutation` for modifications
- Use `queryClient.invalidateQueries` in mutation `onSuccess` callbacks
- Keep query functions pure - extract complex logic to service functions
- Always handle error states with meaningful messages
- Provide proper fallback values to prevent undefined errors
- Organize query files by feature/domain
- **Validate API contracts**: For OCM API bugs, check `ocm-api-model` repo:
  - PATCH methods support partial updates (only send changed fields)
  - Verify required vs optional fields in type definitions
  - Confirm request/response structure in OpenAPI spec

**TypeScript Standards** (`.cursor/rules/typescript-rules.mdc`):
- Define prop types using TypeScript interfaces
- Avoid `any` - use `unknown` if needed
- Use existing types from `src/types/` directories
- Define return types for all functions
- Use `import type` for type-only imports

**PatternFly UI**:
- Use PatternFly components (no custom CSS)
- Use PatternFly utility classes for spacing/layout
- Import from `@patternfly/react-core`, `@patternfly/react-table`, etc.
- **Check PatternFly docs** via Context7 MCP when:
  - Unsure about component props or API
  - Need examples of component usage
  - Looking for best practices or patterns
  - Debugging component behavior

**Using Context7 for PatternFly docs**:
```
Use context7 to query PatternFly React documentation:
- "PatternFly Button component API"
- "PatternFly DataList examples"
- "PatternFly Form validation patterns"
```

Context7 fetches current documentation, ensuring you have up-to-date component APIs and examples.

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
- Check if unit tests exist and suggest adding/modifying if needed

### 5. Run Linters and Formatters

**Check code quality:**
```bash
yarn lint        # ESLint checks (includes Prettier)
yarn typecheck   # TypeScript type checking
```

**Auto-fix formatting issues:**
```bash
yarn prettier:fix   # Fix formatting in all src files
# or
npx lint-staged     # Fix only staged files (same as pre-commit hook)
```

**Important notes:**
- `yarn lint` runs both ESLint and Prettier checks
- Husky pre-commit hook automatically runs `lint-staged` on commit
- Staged files in `src/` are auto-formatted by Prettier on commit
- If auto-fix fails, commit will be blocked - fix manually

Fix any errors before proceeding.

### 6. Test Manually

Run the application locally:
```bash
yarn start  # Proxy mode (manual refresh)
# or
yarn dev    # HMR mode (instant updates)
```

- Follow the reproduction steps
- Verify the bug no longer occurs
- Check that the fix doesn't break related functionality
- Test edge cases
- Verify error states and loading states work correctly

### 7. Generate Implementation Notes

Create `artifacts/bugfix/implementation-{issue-key}.md`:

```markdown
# Implementation: {OCMUI-XXXX}

**Branch:** bugfix/OCMUI-{number}-{description}

## Changes
- `{file}:{lines}` - {what and why}

## Key Decisions
{If applicable: - {Decision}: {rationale}}

## Standards Applied
{Any specific .cursor/rules followed, e.g., "Used React Query pattern for API calls"}

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
- **Check .cursor/rules**: Always reference the appropriate rules files for your changes
- **OCM API validation**: When fixing API-related bugs, reference the `ocm-api-model` repo at `/workspace/repos/ocm-api-model` to validate endpoint behavior, required fields, and request structure
- **PatternFly documentation**: Use Context7 MCP to fetch PatternFly React docs on-demand for component APIs, props, examples, and best practices
