---
name: merge-pull-request
description: Merge a GitHub pull request so the base branch gains exactly one changelog entry per change. Picks rebase, squash, or merge commit from the repository's settings and the PR's history, writes the squash or merge commit message explicitly so no Conventional Commit subject is repeated in a commit body, pins the merge to the reviewed head, and verifies the merged commit afterwards. Use when asked to merge a PR, land a PR, or complete a merge. Do not use to review a PR (use pull-request), to squash its branch before merge (use squash-pull-request), pr-steward uses this skill for its opt-in automerge.
license: MIT
---

# Merge Pull Request

Merge one pull request so its change appears once in the history that changelog and release tooling reads. The merge is one command; the work is choosing the method and writing the commit message GitHub would otherwise write.

## Set the Authority Boundary

Asking to merge a PR authorizes merging that PR with a method the repository allows. It does not authorize `--admin`, merging with failing or pending checks or an unmet review requirement, deleting the branch, or merging any other PR. When a requirement is unmet, report it and stop; `--auto` is acceptable only when the user asked to merge once checks pass.

## Establish the Target

1. Treat a PR number or URL as authoritative; otherwise resolve the PR for the current branch with `gh pr view`.
2. Read the PR: `gh pr view <PR> --json number,title,baseRefName,headRefName,headRefOid,isDraft,mergeable,mergeStateStatus,reviewDecision,statusCheckRollup,commits`. Record `headRefOid`; the merge is pinned to it.
3. Read what the repository allows: `gh repo view --json mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed,deleteBranchOnMerge`.
4. Read the commit convention and the changelog tooling (`release-plz.toml`, `cliff.toml`, `.releaserc*`, `CHANGELOG.md`, `CONTRIBUTING`): they decide what a commit subject must look like. Without such tooling the rules below still hold.

## One Entry Per Change

Conventional Commits changelog tooling turns every line shaped like `type(scope): description` in a commit message into an entry. GitHub's default merge messages put the PR title or the branch's commit subjects into the body of the commit they create, so the same change is counted twice. This merge commit is therefore unacceptable:

```text
Merge pull request #76 from StephanMeijer/test/exercise-update-gym-list

fix: example message
```

The rule: after the merge, the commits added to the base branch carry exactly one Conventional subject per change, and no other line in any of those commits matches `^(feat|fix|docs|refactor|perf|test|build|ci|chore|revert)(\([^)]+\))?!?:` followed by a space. GitHub's repository settings cannot produce a merge commit without the PR title in it, so write the message yourself with `--subject` and `--body`. GitHub silently ignores an empty body and falls back to the default (the PR title or the commit list), so `--body ""` is not enough: pass a non-empty plain-English body that matches no Conventional pattern, such as one sentence saying what the change does.

## Pick the Method

In this order, the first the repository allows and the PR fits:

- **Rebase** (`--rebase`) when every commit on the branch is one Conventional commit that should be its own entry and the repository does not need signed history on the base branch. Nothing is written, but GitHub always rewrites rebased commits and drops their signatures; when signed history matters, use the next method that fits instead: squash for one change, a merge commit when each commit should stay an entry, which keeps their signatures; GitHub signs the squash or merge commit. A PR that should be one entry but has several commits goes through `squash-pull-request` first, then rebase-merges.
- **Squash** (`--squash`) when the PR is one change and its branch history is not worth keeping. The subject is the Conventional subject for the net change, written per the `git-commit` skill from the diff, not copied from the PR title (titles are plain English). The body is empty: GitHub's default body lists every squashed commit subject, which is the double entry.
- **Merge commit** (`--merge`) when the repository or the user prefers merge commits, or the branch's Conventional commits should each stay an entry. The subject stays GitHub's non-Conventional `Merge pull request #<N> from <owner>/<branch>` so tooling ignores it, and the body is one plain sentence (see above). Never set the PR title as the merge subject or body: a Conventional title makes the merge commit itself an entry.

## Merge

```bash
gh pr merge <PR> --rebase --match-head-commit <headRefOid>
gh pr merge <PR> --squash --match-head-commit <headRefOid> --subject "<type>(<scope>): <description>" --body "<plain sentence>"
gh pr merge <PR> --merge  --match-head-commit <headRefOid> --subject "Merge pull request #<N> from <owner>/<branch>" --body "<plain sentence>"
```

`--match-head-commit` makes GitHub refuse the merge if the branch moved after you read it. Add `--delete-branch` only when the user asked for it or the repository deletes branches on merge anyway. Never pass `--admin`.

### Stacked PRs

A PR in a native GitHub stack (`gh api repos/<owner>/<repo>/pulls/<N> --jq .stack` is not null) cannot be merged with `gh pr merge` ("must be merged using the asynchronous merge REST API"). `gh stack merge` cannot set the message, so call the API directly, one PR at a time from the bottom of the stack:

```bash
uuid=$(gh api -X PUT repos/<owner>/<repo>/pulls/<N>/merge-async -f merge_method=<merge|squash|rebase> \
  -f merge_action=direct_merge -f sha=<headRefOid> -f commit_title="<subject>" -f commit_message="<plain sentence>" --jq .details.uuid)
gh api repos/<owner>/<repo>/pulls/<N>/merge-async/$uuid   # poll until status is not "pending"; expect "merged"
```

Merging a PR this way also merges every open PR below it, so never merge a higher PR to land a lower one. After each merge GitHub retargets the next PR to the base branch and usually rewrites its commits, so its head SHA changes. Do not re-pin blindly: re-pin only when `git rev-parse <new-head>^{tree}` equals the tree of the head that was reviewed (fetch first); otherwise stop and get the new head reviewed.

## Verify

```bash
git fetch origin <base>
git log --format='%H%n%B' <base-before>..origin/<base> | grep -cE '^(feat|fix|docs|refactor|perf|test|build|ci|chore|revert)(\([^)]+\))?!?: '
```

When the repository expects signed history, check the signature of every commit the merge added and report any that is not verified:

```bash
gh api repos/<owner>/<repo>/commits/<sha> --jq '.commit.verification | "\(.verified) \(.reason)"'
```

`<base-before>` is the base branch's SHA recorded before the merge. The count must equal the number of changes the PR represents: one for a squash, one per branch commit for a rebase, and for a merge commit the branch commits alone. If the merge or squash commit came back with a body GitHub filled in, say so in the report; do not rewrite the base branch to fix it.

Confirm `gh pr view <PR> --json state,mergeCommit` shows `MERGED` and the expected commit.

## Report

The PR, the method and why, the exact subject and body written, the merge commit SHA, the grep count against the expected count, whether the branch was deleted, and anything that blocked or was skipped. Never describe the PR as merged without the provider showing `MERGED`.
