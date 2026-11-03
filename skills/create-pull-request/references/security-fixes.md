# Undisclosed Security Fixes

Use this workflow only when the change fixes a vulnerability that has not already been disclosed.
An ordinary bug with no sensitive exploit information follows the normal PR path.

## Establish the Private Channel

1. Read the repository security policy and follow its reporting route.
2. Determine whether a private vulnerability report or draft repository security advisory already
   owns the work. Do not infer disclosure from the absence of a public issue.
3. If no private channel exists, stop before any push or PR. Ask the user to authorize a private
   report or maintainer contact; creating a PR does not authorize creating an advisory.
4. Confirm the current user is an advisory collaborator and identify the advisory's temporary
   private fork and base branch from provider data.

Never push an undisclosed fix to a public fork, public branch, or ordinary pull request. Never fall
back to a public route when private-fork access or advisory permissions fail.

Official GitHub references:

- <https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/report-privately>
- <https://docs.github.com/en/code-security/tutorials/fix-reported-vulnerabilities/collaborate-in-a-fork>

## Prepare the Patch

- Keep one independent vulnerability or shared root cause per branch and PR.
- Use a neutral branch name and concise commit subject that do not reveal an exploit before
  disclosure.
- Include a regression test that fails on the vulnerable revision for the intended reason, the
  smallest centralized fix, and verification of equivalent paths.
- Follow repository-specific human review, signing, draft, and readiness requirements even though
  the PR is private.
- Run the complete local gate. Temporary private forks may be invisible to ordinary integrations and
  hosted CI, so list every unavailable check explicitly.

## Open and Verify Privately

Push only to the temporary private fork. Create the PR from the advisory's collaboration area or
against the temporary fork branch identified by the provider. Open it as a draft when the repository
requires human review or remediation is still being coordinated.

After opening, verify:

- the base and head both belong to the private remediation workspace;
- the PR, branch, commits, logs, and attachments reveal nothing publicly;
- the advisory links the patch and the intended collaborators can access it;
- no public issue, PR, release note, or workflow run was created accidentally.

Only the maintainer decides when to merge, release, request a CVE, and publish the advisory. Do not
convert the private PR into a public PR unless the maintainer explicitly coordinates that transition.
