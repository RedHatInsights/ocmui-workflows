---
name: list-unscrubbed
description: List all unscrubbed bugs from OCMUI Jira for Interruption Catcher duties
---

# /list-unscrubbed - List Unscrubbed Bugs

## Purpose

Fetch and display all unscrubbed bugs from the OCMUI Jira project. This is the starting point for Interruption Catcher duties - it shows you what bugs need scrubbing.

## ⚠️ Known Limitation: CVE/Vulnerability Tickets

**CVE and Vulnerability issue types are NOT included in the results** due to API security restrictions. The Jira MCP integration cannot access Vulnerability tickets even when they are marked as public.

**Workaround:** Manually check the Jira dashboard for CVE tickets at the start of IC duty. CVE tickets will have a lock icon (🔒) and summaries starting with "CVE-YYYY-NNNNN".

For CVE handling instructions, see `bugfix/reference/CVE-HANDLING.md`.

## Process

### 1. Execute JQL Query

Use `mcp__mcp-atlassian__jira_search` with the following query:

```jql
project = OCMUI and type in (Bug, Vulnerability) and status not in (Done, Closed) and (labels not in (scrubbed) or labels is EMPTY) and status = "To Do" order by priority, created desc
```

**Note:** The query includes `Vulnerability` type for completeness, but the API will filter these out due to security restrictions.

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

**Order:** Display in the order returned by JQL (by priority, then creation date desc)

### 3. Provide Summary

After the table, show:
- Total number of unscrubbed bugs found
- Breakdown by priority (if any Blocker/Critical, highlight them)
- **⚠️ Reminder:** "Note: CVE/Vulnerability tickets are not included in this list. Check the Jira dashboard manually for CVE tickets."

### 4. Next Steps Guidance

Tell the user:

```
To scrub a specific bug, run:
  /scrub OCMUI-1234

Or provide a Jira URL:
  /scrub https://issues.redhat.com/browse/OCMUI-1234

⚠️ Don't forget to manually check for CVE tickets in the Jira dashboard!
```

**Do not automatically launch into scrubbing.** Just show the list and wait for the user to decide.

## Output

- Console output only (no artifact file created)
- Clean table of unscrubbed bugs
- Brief next-step instructions
- CVE limitation reminder

## Success Criteria

After running this skill:
- [ ] Fetched current list of unscrubbed bugs from Jira
- [ ] Displayed them in a readable table format
- [ ] Showed summary statistics
- [ ] Provided clear next-step guidance
- [ ] Reminded user about CVE ticket limitation
- [ ] User knows how to proceed with scrubbing

## Notes

- **No dashboard URLs**: This skill uses JQL directly, which is more reliable than dashboard bookmarks
- **Real-time data**: Every time you run this, it fetches fresh data from Jira
- **Interruption Catcher workflow**: This is step 1 of IC duty - run this first to see what needs attention
- **Empty results**: If no unscrubbed bugs are found, congratulate the user - everything is scrubbed!
- **CVE tickets excluded**: Due to security restrictions in the Jira API, CVE/Vulnerability tickets cannot be accessed via the MCP integration. This is intentional and cannot be worked around. Always check the Jira dashboard manually for CVE tickets.
