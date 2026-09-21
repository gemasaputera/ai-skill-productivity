---
name: jira-subtask-creator
description: "Create a Jira sub-task under an existing parent issue in any project, with optional custom fields, assignee, and status transitions discovered from Jira metadata."
---

# Jira Sub-task Creator

Create a Jira sub-task under an existing parent issue. Works with any Jira project — project key, issue type, assignee, custom fields, and workflow status are provided by the user or discovered from Jira rather than hardcoded.

## When to Use

- User asks to create a Jira sub-task
- User wants to break a story, epic, or task into smaller work items
- User wants to add a child task / follow-up under an existing issue

## Inputs

**Required**
- **Parent issue key** (e.g., `ABC-123`) — or inferred from recent context
- **Summary** — the title of the sub-task

**Optional**
- **Description** — free-form; supports headings and bullet lists
- **Assignee**
- **Priority**, **labels**, **components**, **due date**, **story points**
- **Custom fields** — squad, team, dates, or any org-specific field
- **Target status** — transition the new sub-task after creation

The project is normally inferred from the parent issue key prefix, so the user rarely needs to specify it.

## Configuration

These defaults are optional. A team can pin them once; otherwise resolve per task or ask the user. Leave any value blank to discover it from Jira.

| Setting | Default | Notes |
|---------|---------|-------|
| `project_key` | inferred from parent | Prefix of the parent issue key |
| `issue_type` | `Sub-task` | Falls back to `Task` if the project has no sub-task type |
| `assignee` | unassigned | Resolve a display name to an account/user ID before assigning |
| `custom_fields` | none | Look up field IDs and option IDs via Jira metadata (see below) |
| `target_status` | leave as created | Transition only when the user asks |
| `jira_base_url` | from the Jira instance | Used to build the browse link in the confirmation |

## Workflow

### Step 1: Gather Required Inputs

Ask only for what is missing:
- Parent issue key
- Summary

If the user says "same parent as last time", reuse the previous parent key. If the parent is unknown, ask — do not guess.

### Step 2: Resolve Optional Fields

Apply any overrides the user provides (assignee, priority, labels, custom fields, target status). For values not supplied, use the configuration defaults or leave them unset. Do not block creation on optional fields.

### Step 3: Discover Field IDs (do not hardcode)

Custom field IDs differ per Jira instance. Resolve them at runtime instead of assuming:

1. Fetch the create metadata for the target project and issue type to list available fields.
2. Match fields by their **display name** (e.g., "Squad", "WBS Start Date").
3. For select fields, fetch the allowed values and match by name to get the option **ID**.
4. Confirm the match with the user if the field name is ambiguous.

If the metadata is unavailable, ask the user for the field ID or skip that field.

### Step 4: Draft a Description (optional)

If the user provides no description, you may draft one from available context:
- Recent commits on the relevant branch: `git log <branch> --oneline -5`
- The parent issue's description or acceptance criteria

Keep it short and factual. For richer descriptions use headings or Jira sections, e.g.:

```
h3. Context
h3. Root Cause / Scope
h3. Acceptance Criteria
h3. Notes
```

Never invent requirements — only summarize what the context actually says.

### Step 5: Create the Sub-task

Use the Jira MCP tool to create the issue:

```
jira_create_issue:
  project_key: "<project>"
  summary: "<summary>"
  issue_type: "<issue_type>"
  description: "<description>"
  additional_fields: {
    "parent": "<parent-key>",
    "<custom_field_id>": "<value>"
  }
```

Include only the fields the user requested or that the project requires.

### Step 6: Assign and Transition (optional)

- Assign only if the user provides an assignee (resolve names to IDs first).
- Transition only if the user requests a target status; fetch available transitions before moving the issue.

### Step 7: Confirm

Report back with:
- Issue key and clickable URL
- Parent issue key
- Assignee, status, and any custom fields that were set
- A note about anything skipped or left unset

## Example

**User**: "Create a sub-task under ABC-123 summarizing 'Fix settlement balance rounding'"

**Agent actions**:
1. Infer project `ABC` from the parent key, type `Sub-task`.
2. Create the sub-task under `ABC-123`.
3. Leave assignee/status unset unless requested.
4. Report: "Created ABC-456: Fix settlement balance rounding — https://<jira-host>/browse/ABC-456"

**User**: "Same as last time, plus assign it to me and set the squad to Platform."

**Agent actions**:
1. Reuse the previous parent, type, and custom fields.
2. Resolve "squad" → field ID + option ID via metadata.
3. Resolve the user → assignee ID, assign, then confirm.

## Notes

- Never hardcode project keys, custom field IDs, option IDs, hosts, or assignees — discover or ask.
- Prefer the parent issue's project and reuse prior context when the user references "last time".
- Ask before guessing when a required value (parent, summary) is unknown.
- Leave missing optional data unset rather than inventing values.
