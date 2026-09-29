# Follow-Up Issues

Open a follow-up issue for every out-of-scope finding (see the scope rule in `SKILL.md`), and for other work a review or fix surfaces that is logical to track but out of scope for the PR: a pre-existing bug, a code-versus-spec gap, a flaky test, documentation debt, or a deferred feature. Do not file speculative wishes. Delegate filing to one sub-agent per batch. When the `github-issue` skill is installed, that sub-agent should follow it.

## For Each Issue

1. **Deduplicate.** Search open and closed issues (`gh issue list --state all --search "<keywords>"`). If one already covers it, record `covered by #N` instead of filing.
2. **Cite the origin.** Open the body with `Out of scope for #<PR> (<PR title>), raised in <review or thread URL>.` If the PR closes an issue, add `which closes #<issue>`. Find the exact comment URL through the PR's reviews, review comments, and issue comments.
3. **Write a precise body.** Problem (file, symbol, trigger scenario), proposed fix, and acceptance criteria. Confirm names against the base branch with `git show origin/<base>:<path>`; do not invent facts. Match the style of recent issues in the repository.
4. **State relationships natively.** Text mentions are not enough.
   - Numeric ID: `gh api repos/<owner>/<repo>/issues/<n> --jq .id`.
   - **Blocked by**, when the follow-up cannot start until the origin lands: `gh api -X POST repos/<owner>/<repo>/issues/<new>/dependencies/blocked_by -F issue_id=<blocker id>`. Use the origin PR's closing issue as the blocker; try the PR itself first when that link is more precise and fall back if the API refuses it.
   - **Sub-issue**, when a parent tracking issue fits: `gh api -X POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<new id>`.
   - Read each relationship back to verify it.
5. **Set the issue type** when the repository uses types: list them through GraphQL `repository { issueTypes(first: 20) { nodes { id name } } }`, then `updateIssue(input: {id, issueTypeId})`.
6. **Close the loop on the PR.** When the finding came from a review thread, reply in that thread with `Out of scope, filed as #<new>.` and resolve it. Otherwise post one short top-level comment: `Out of scope, filed as #<new> — <title>`. Skip the PR comment when the PR is closed or excluded in the state file; the issue body alone cites it.
7. **Record it** in the steward's `STATE.md` so no later tick files it again.

## Report

Return a table: candidate, issue URL (or `covered by #N` / `skipped: <reason>`), type, relationships set and verified, origin comment URL, PR comment URL.
