---
name: jira-issue-report
description: "Generate Jira issue reports from Atlassian MCP tools with parent story and epic enrichment."
---

# Jira Issue Report Generator

Generate Jira issue reports from Atlassian MCP tools with parent story and epic enrichment.

## When to Use

- User asks for a Jira issue report
- User wants issues listed in a table with epic/parent enrichment
- User asks for sprint reports or squad-based issue lists
- User mentions exporting Jira data to CSV

## Requirements

1. Ask clarifying questions only when required inputs are missing.
2. Search issues with JQL based on requested filters.
3. Default scope: all accessible projects unless user specifies project keys.
4. Support date filtering by Created and/or Updated with explicit timezone.
5. Remove duplicates by issue key.
6. Sort by user-requested date field and direction.

## Output

Return both:
- **Markdown table** in chat
- **CSV file** in current workspace

## Columns

| Column | Notes |
|--------|-------|
| Title | Issue summary |
| Type | Only Story/Sub-task, else empty |
| Jira Link | Clickable link |
| Create Date | Readable format (WIB when +0700) |
| Reported By | Reporter display name |
| Assigned To | Assignee display name |
| Sprint Name | From customfield_10104 |
| Parent Story | For sub-tasks only — parent issue key |
| Parent Story Epic Link | Epic of the parent story |

## Field Mapping

| Field Name | Custom Field ID |
|------------|----------------|
| Epic Link | customfield_10100 |
| Sprint | customfield_10104 |

## Enrichment Logic

1. If issue type is **Sub-task**, resolve parent issue key.
2. Resolve parent issue's epic key.
3. Resolve epic summary and target dev date.
4. Leave empty when data is missing.

## Workflow

### Step 1: Gather Filters

Ask user for:
- **Project key(s)** (e.g., NQLA) — or use all accessible
- **Date filter** — Created or Updated, with date range
- **JQL extras** — status, assignee, sprint, etc.
- **Sort** — field and direction

### Step 2: Build JQL

Combine filters into a JQL query. Examples:
- `project = NQLA AND created >= "2026-07-01" ORDER BY created ASC`
- `project = NQLA AND updated >= "2026-07-28" AND updated <= "2026-08-04" ORDER BY updated ASC`

### Step 3: Search Issues

```
jira_search:
  jql: "<built-jql>"
  fields: "*all"
  limit: 50
```

Paginate if more results exist.

### Step 4: Enrich Each Issue

For each issue:
1. Check if type is "Sub-task" → get parent key
2. If has parent, fetch parent issue → get parent's Epic Link
3. If has epic, fetch epic → get Epic Name and Tanggal Target Selesai Dev

Use `jira_get_issue` for parent/epic lookups.

### Step 5: Deduplicate

Remove duplicate issues by issue key.

### Step 6: Format Output

**Markdown table:**
- Use clickable Jira links
- Dates in readable format (WIB when +0700)
- Mention total rows returned

**CSV file:**
- Write to workspace as `jira-report-<date>.csv` & `jira-report-<date>.md`
- Include all columns in stable order

### Step 7: Present Results

Show the markdown table in chat and confirm CSV and Markdown file locations.

## Notes

- Preserve clickable Jira links in markdown output
- Use stable column order as listed above
- Keep date display readable (WIB when +0700 applies)
- Mention total rows returned at the end