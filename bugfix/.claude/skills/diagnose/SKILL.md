---
name: diagnose
description: Root cause analysis for bugs
---

# /diagnose - Root Cause Analysis

## Purpose

Perform systematic root cause analysis to understand WHY the bug occurs, not just WHAT happens. This guides the fix implementation.

## Prerequisites

- Bug has been scrubbed
- Access to uhc-portal codebase
- Understanding of the reported issue

## Process

### 1. Review Bug Information

Understand from Jira or scrub report:
- Exact steps to trigger the bug
- Observable symptoms
- Expected vs actual behavior
- Environment conditions

### 2. Locate Relevant Code

Use file navigation to find the affected code:
- Search for component names from the bug (use Grep)
- Look for error messages in the codebase (use Grep)
- Find files related to the feature area (use Glob)
- Check routing to understand page flow

### 3. Trace Execution Flow

Follow the code path:
- Identify the UI component where the bug manifests
- Trace data flow: props → state → API calls
- Check Redux/React Query state management
- Follow event handlers and callbacks
- Examine form validation logic if applicable

### 4. Validate API Contracts (if API-related bug)

**When bug involves OCM API calls** (clusters_mgmt, accounts_mgmt, addons_mgmt, etc.):

Check the API model to validate assumptions about the endpoint:

**Find endpoint definition**:
```bash
# Model files define API structure
ls /workspace/repos/ocm-api-model/model/clusters_mgmt/v1/*{feature}*.model

# Example: cluster update
# - cluster_resource.model (defines methods: Get, Update, Delete)
# - cluster_type.model (defines Cluster object fields)
```

**Key questions to answer**:
1. **HTTP Method**: GET, POST, PATCH, PUT, or DELETE?
   - `method Update` typically = PATCH (supports partial updates)
   - `method Add` typically = POST (requires complete object)

2. **Required vs Optional Fields**: Check the type definition
   - Fields without defaults are typically required
   - Check `openapi.json` for explicit `required` array

3. **Field Types**: Validate data types and structure
   - `String`, `Boolean`, `Integer`, etc.
   - Nested objects (e.g., `Proxy`, `AWS`)

**OpenAPI Specification** (generated from model):
```bash
# Full OpenAPI spec with request/response schemas
cat /workspace/repos/ocm-api-model/openapi/{service}/v1/openapi.json
```

**Common API validation scenarios**:
- ✅ PATCH endpoints support **partial updates** (only send changed fields)
- ✅ Check if fields are optional before omitting them
- ✅ Validate enum values and data types
- ✅ Understand link fields vs embedded objects

**Example from OCMUI-4183**:
```markdown
Bug: Sending empty proxy object when only CA cert changed
API Validation:
- cluster_resource.model: method Update (= PATCH method)
- cluster_type.model: Proxy is optional field
- openapi.json: `required: []` (no required fields)
Conclusion: PATCH supports partial updates, only send changed fields ✅
```

**When to validate API contracts**:
- Sending unexpected data to API
- Getting API errors (400, 422 validation errors)
- Unsure about required fields
- Payload structure questions


### 5. Check PatternFly Documentation (if UI component bug)

**When bug involves PatternFly components** (Button, DataList, Form, Modal, Table, etc.):

Use Context7 MCP to fetch current PatternFly React documentation:

**Query PatternFly docs for**:
- Component props and API reference
- Usage examples and patterns
- Best practices and accessibility guidelines
- Known issues or breaking changes

**Example queries**:
```
Use context7 to query:
- "PatternFly Button component props and variants"
- "PatternFly DataList component examples"
- "PatternFly Form validation patterns"
- "PatternFly Modal accessibility requirements"
```

**When to check PatternFly docs**:
- Component not rendering as expected
- Props not working as documented in old notes
- Looking for correct prop names or types
- Need examples of proper component usage
- Accessibility issues with PatternFly components
- Migration between PatternFly versions

**Benefits**:
- Get current, up-to-date documentation
- Find official examples and patterns
- Avoid deprecated props or patterns
- Ensure accessibility compliance


### 6. Examine Git History

Understand recent changes:
```bash
git log --oneline --since="30 days ago" -- {affected-file}
git blame {affected-file}
```

Look for:
- Recent changes to the affected code
- Related PRs that might have introduced the bug
- Commits around the time the bug was first reported

### 7. Identify Root Cause

Test hypotheses:
- Add console.log or debugger statements
- Run the reproduction steps
- Examine the actual values and execution flow
- Confirm the actual root cause

### 8. Recommend Fix Strategy

Based on the root cause, propose how to fix it:
- **Minimal change**: What's the smallest fix?
- **Correct approach**: What's the right way to solve this?
- **Breaking changes**: Will this affect existing behavior?

### 9. Generate Root Cause Analysis

Create `artifacts/bugfix/root-cause-{issue-key}.md`:

```markdown
# Root Cause: {OCMUI-XXXX}

## Cause
{1-2 sentence statement of the problem}

## Affected Code
- `{file}:{line}` - {issue description}

## Fix Strategy
{Approach to fix}

**Changes needed:**
- {file} - {what needs to change}

**Confidence:** High/Medium/Low - {why}

## Next Step
**Issue:** OCMUI-{XXXX}
**Ready for:** /fix OCMUI-{XXXX}
```

### 10. Re-read Controller and Return

After generating the root cause analysis, re-read `.claude/skills/controller/SKILL.md` and return control to the controller for next step recommendations.

## Output

- `artifacts/bugfix/root-cause-{issue-key}.md` — Root cause analysis
- Console output with next step: `/fix {issue-key}`

## Notes

- **Use file:line notation**: Always reference code locations as `src/components/ClusterList.tsx:245`
- **Test hypotheses systematically**: Don't just guess - add logging and verify
- **Be thorough**: Root cause analysis quality directly affects fix quality
- **PatternFly docs**: For UI component bugs, use Context7 MCP to fetch current PatternFly React documentation
