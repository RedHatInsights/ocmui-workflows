---
name: list-unscrubbed
description: List all unscrubbed bugs from OCMUI Jira for Interruption Catcher duties
---

# /list-unscrubbed - List Unscrubbed Bugs

## Purpose

Fetch and display all unscrubbed bugs from the OCMUI Jira project. This is the starting point for Interruption Catcher duties - it shows you what bugs need scrubbing.

## Process

### 1. Execute JQL Query

Use `mcp__mcp-atlassian__jira_search` with the following query:

```jql
project = OCMUI and (type in (Bug, Vulnerability) OR summary ~ "CVE-*") and status not in (Done, Closed) and (labels not in (scrubbed) or labels is EMPTY) and status = "To Do" order by priority, created desc
```

**Note:** The `OR summary ~ "CVE-*"` clause is a workaround for Vulnerability tickets that may be misreported by the API. CVE tickets typically have "CVE-YYYY-NNNNN" in their summary.

**Parameters:**
- **fields**: `summary,priority,status,labels,assignee,created,updated,reporter,description,issuetype`
- **limit**: 20
- **start_at**: 0

### 2. Display Results

Format the results in a clean, readable table showing:

| Key | Priority | Summary | Assignee | Created | Labels |
|-----|----------|---------|----------|---------|--------|
| OCMUI-1234 | Major | Bug title here... | Unassigned | 2026-03-15 | ui-feature:networking |

**Include:**
- Issue key (linkable format: `https://issues.redhat.com/browse/{key}`)
- Priority level
- Summary (truncate if very long)
- Assignee (show "Unassigned" if none)
- Created date (just the date, not time)
- Relevant labels (especially priority suggestions, feature tags, product tags)

**Order:** Display in the order returned by JQL (newest first, then by priority)

### 3. Provide Summary

After the table, show:
- Total number of unscrubbed bugs found
- Breakdown by priority (if any Blocker/Critical, highlight them)

### 4. Next Steps Guidance

Tell the user:

```
To scrub a specific bug, run:
  /scrub OCMUI-1234

Or provide a Jira URL:
  /scrub https://issues.redhat.com/browse/OCMUI-1234
```

**Do not automatically launch into scrubbing.** Just show the list and wait for the user to decide.

## Output

- Console output only (no artifact file created)
- Clean table of unscrubbed bugs
- Brief next-step instructions

## Success Criteria

After running this skill:
- [ ] Fetched current list of unscrubbed bugs from Jira
- [ ] Displayed them in a readable table format
- [ ] Showed summary statistics
- [ ] Provided clear next-step guidance
- [ ] User knows how to proceed with scrubbing

## Notes

- **No dashboard URLs**: This skill uses JQL directly, which is more reliable than dashboard bookmarks
- **Real-time data**: Every time you run this, it fetches fresh data from Jira
- **Interruption Catcher workflow**: This is step 1 of IC duty - run this first to see what needs attention
- **Empty results**: If no unscrubbed bugs are found, congratulate the user - everything is scrubbed!
- **CVE workaround**: The query includes `summary ~ "CVE-*"` to catch Vulnerability tickets that may be misidentified by the MCP API
