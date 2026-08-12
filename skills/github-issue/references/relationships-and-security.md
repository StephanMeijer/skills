# Relationships and Security

Use issue URLs for cross-repository targets and numbers for same-repository targets. Inspect both ends before creating a relationship.

## Parent and Sub-Issue Hierarchy

Create a sub-issue under a parent:

```bash
gh issue create --repo OWNER/REPO \
  --title "TITLE" --body-file BODY_FILE \
  --parent PARENT_NUMBER
```

Set, change, or remove the parent of an existing issue:

```bash
gh issue edit CHILD_NUMBER --repo OWNER/REPO --parent PARENT_NUMBER
gh issue edit CHILD_NUMBER --repo OWNER/REPO --remove-parent
```

Manage a parent's sub-issues directly:

```bash
gh issue edit PARENT_NUMBER --repo OWNER/REPO --add-sub-issue CHILD_NUMBER
gh issue edit PARENT_NUMBER --repo OWNER/REPO --remove-sub-issue CHILD_NUMBER
```

A child has one parent. Use hierarchy for decomposition, not merely because two issues mention the same area.

## Blocking Dependencies

On creation:

```bash
gh issue create --repo OWNER/REPO \
  --title "TITLE" --body-file BODY_FILE \
  --blocked-by BLOCKER_NUMBER \
  --blocking BLOCKED_NUMBER
```

On an existing issue:

```bash
gh issue edit NUMBER --repo OWNER/REPO \
  --add-blocked-by BLOCKER_NUMBER \
  --add-blocking BLOCKED_NUMBER

gh issue edit NUMBER --repo OWNER/REPO \
  --remove-blocked-by OLD_BLOCKER_NUMBER \
  --remove-blocking OLD_BLOCKED_NUMBER
```

Creating dependencies requires at least triage permission. GitHub limits each direction to 50 linked issues. Verify with:

```bash
gh issue view NUMBER --repo OWNER/REPO --json blockedBy,blocking
```

Official reference: <https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/creating-issue-dependencies>

## Duplicates

Close an issue as a duplicate only when the user asked for closure and the canonical issue is verified:

```bash
gh issue close DUPLICATE_NUMBER --repo OWNER/REPO \
  --reason duplicate \
  --duplicate-of CANONICAL_NUMBER
```

Do not use a duplicate relationship for merely overlapping work.

## Related Issues

**Relates to** is native GitHub relationship metadata. As of GitHub CLI 2.97.0 and the public GitHub APIs available in August 2026, neither `gh issue` nor the documented REST and GraphQL APIs expose a write operation for it.

Confirm that the installed CLI has not added support:

```bash
gh issue edit --help
```

If no related-issue flag exists, do not modify the issue. Report that the native relationship remains unapplied because GitHub has not exposed it to this CLI-only workflow. Do not add a body reference, open a browser, call undocumented internal endpoints, or substitute `blocked by`, `blocking`, or parent/sub-issue metadata.

## Link a Code-Scanning Security Alert

GitHub's code-scanning alert-to-issue tracking is a public-preview feature. Each code-scanning alert can link to one issue; an issue can track up to 50 alerts. Statuses do not synchronize: closing the alert does not close the issue, and closing the issue does not close the alert.

As of GitHub CLI 2.97.0 and the public GitHub APIs available in August 2026, `gh issue` and the documented REST and GraphQL APIs do not expose an operation to create this link. Verify both resources and the installed CLI capability:

```bash
gh api "repos/ALERT_OWNER/ALERT_REPO/code-scanning/alerts/ALERT_NUMBER"
gh issue view ISSUE_NUMBER --repo ISSUE_OWNER/ISSUE_REPO --json number,url,title,state
gh issue edit --help
```

If no security-alert relationship flag exists, do not modify either resource. Report that the native link remains unapplied because GitHub has not exposed it to this CLI-only workflow. Do not open a browser or call undocumented internal endpoints. Linking requires write access to the alert repository when GitHub exposes a supported command or API.

Official references:

- <https://docs.github.com/en/code-security/concepts/code-scanning/code-scanning-alert-tracking-using-issues>
- <https://docs.github.com/en/code-security/how-tos/manage-security-alerts/manage-code-scanning-alerts/track-alerts-in-issues>

Do not put the literal secret from a secret-scanning alert into an issue. The documented issue-tracking relationship currently applies to code-scanning alerts, not Dependabot or secret-scanning alerts.

## Route Private Vulnerability Reports

Never file non-public vulnerability details in a public issue.

For an external reporter, use private vulnerability reporting when the repository has enabled it. This creates a private report for maintainers:

```bash
gh api --method POST \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/security-advisories/reports" \
  --input REPORT_JSON
```

For a repository administrator or security manager, create a draft repository security advisory instead:

```bash
gh api --method POST \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  "repos/OWNER/REPO/security-advisories" \
  --input ADVISORY_JSON
```

Both JSON documents require `summary` and `description`. A draft advisory also requires `severity` and a `vulnerabilities` array; a private report accepts optional vulnerabilities and either `severity` or a CVSS vector. Validate the full payload against the current API documentation before sending it. These are separate security objects, not ordinary issues or code-scanning alert links, and they require Repository security advisories write permission.

Official references:

- <https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/report-privately>
- <https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/fix-reported-vulnerabilities/create-repository-advisory>
- <https://docs.github.com/en/rest/security-advisories/repository-advisories>
