# Native Issue Metadata

Use `--repo OWNER/REPO` consistently. Resolve the actual repository before discovery or writes.

## Inspect Available Values

Assignees must be eligible for the repository:

```bash
gh api --paginate "repos/OWNER/REPO/assignees" --jq '.[].login'
```

List labels and milestones:

```bash
gh label list --repo OWNER/REPO --limit 100
gh api --paginate "repos/OWNER/REPO/milestones?state=all" \
  --jq '.[] | [.number, .title, .state] | @tsv'
```

Issue types are configured by an organization. A personal repository has no organization issue types.

```bash
gh api graphql \
  -f query='query($owner:String!){organization(login:$owner){issueTypes(first:100){nodes{name description isEnabled}}}}' \
  -F owner=OWNER \
  --jq '.data.organization.issueTypes.nodes[] | select(.isEnabled)'
```

List projects owned by the repository owner and inspect a selected project's fields:

```bash
gh project list --owner OWNER
gh project field-list PROJECT_NUMBER --owner OWNER
```

Adding an issue to a project requires the `project` authentication scope. Check before changing authentication:

```bash
gh auth status
```

If the needed scope is absent, explain that `gh auth refresh -s project` requires the user's interactive authorization. Do not broaden token scopes silently.

## Set Standard Issue Metadata

Create with native metadata:

```bash
gh issue create --repo OWNER/REPO \
  --title "TITLE" --body-file BODY_FILE \
  --assignee LOGIN --label "LABEL" --type "TYPE" \
  --milestone "MILESTONE" --project "PROJECT"
```

Flags may be repeated or accept comma-separated values where `gh issue create --help` says so. Quote names containing spaces.

Edit standard metadata:

```bash
gh issue edit NUMBER --repo OWNER/REPO \
  --add-assignee LOGIN --remove-assignee OLD_LOGIN \
  --add-label "LABEL" --remove-label "OLD_LABEL" \
  --type "TYPE" --milestone "MILESTONE" \
  --add-project "PROJECT" --remove-project "OLD_PROJECT"
```

Use `--remove-type` or `--remove-milestone` when the user explicitly asks to clear those fields. `@me` is valid for self-assignment; `@copilot` is supported on GitHub.com but not GitHub Enterprise Server.

Labels, milestones, projects, and their configuration are separate remote resources. Do not create or rename them as a side effect of filing an issue.

## Set Organization-Level Issue Fields

Issue fields appear directly on issues and are distinct from project fields. They are available only for organization-owned repositories. List definitions and current values:

```bash
gh api -H "X-GitHub-Api-Version: 2026-03-10" \
  "orgs/OWNER/issue-fields"
gh api --paginate -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/issues/NUMBER/issue-field-values"
```

Add or update named values without replacing unmentioned fields. `FIELD_ID` is the numeric `id` from the organization endpoint. The value must match the field type; select values use an existing option name.

```bash
gh api --method POST \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/issues/NUMBER/issue-field-values" \
  --input ISSUE_FIELD_VALUES_JSON
```

Use this shape for `ISSUE_FIELD_VALUES_JSON`:

```json
{
  "issue_field_values": [
    {"field_id": 123, "value": "High"}
  ]
}
```

Keep numeric field values as JSON numbers, not quoted strings. Use existing option names for single-select and multi-select fields, and ISO 8601 dates for date fields.

Delete one field value:

```bash
gh api --method DELETE \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/issues/NUMBER/issue-field-values/FIELD_ID"
```

Avoid `PUT .../issue-field-values` unless the user explicitly intends to replace every existing issue field value; it clears values omitted from the request. Setting values requires push access and may trigger notifications.

Official references:

- <https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/adding-and-managing-issue-fields>
- <https://docs.github.com/en/rest/orgs/issue-fields>
- <https://docs.github.com/en/rest/issues/issue-field-values>

## Set Project Fields

First ensure the issue is in the project:

```bash
gh issue edit NUMBER --repo OWNER/REPO --add-project "PROJECT"
```

Then set one project field per invocation. The project owner may differ from the issue repository owner.

```bash
gh project item-edit PROJECT_NUMBER \
  --owner PROJECT_OWNER \
  --url "https://github.com/OWNER/REPO/issues/NUMBER" \
  --field "Status" \
  --value "In Progress"
```

Use the typed flags shown by `gh project item-edit --help` for fields that cannot use a name/value pair, including `--date`, `--number`, `--text`, and `--iteration-id`. Clear a field with node IDs plus `--clear` when required by the installed CLI.

Verify project membership and field values:

```bash
gh project item-list PROJECT_NUMBER --owner PROJECT_OWNER --format json
```

Do not treat a project Status, Priority, or Target date as the same field as an organization-level issue field with the same visible name.
