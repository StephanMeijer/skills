# GitHub PR author workflow

Use authenticated `gh` commands from a checkout whose `origin` points at GitHub.

## Resolve and inspect the PR

Use an explicit PR number when supplied. Otherwise resolve the current branch PR:

```bash
gh pr view --json number -q .number
```

Collect the PR state and patch without switching branches:

```bash
gh pr view <PR> --json number,title,state,isDraft,url,author,baseRefName,headRefName,headRefOid,headRepositoryOwner,isCrossRepository,body,mergeable,mergeStateStatus,reviewDecision
gh pr diff <PR>
gh pr checks <PR> --json bucket,completedAt,description,event,link,name,startedAt,state,workflow
gh api 'repos/<owner>/<repo>/pulls/<PR>/files?per_page=100' --paginate
```

Record `headRefOid` before collecting other surfaces and refetch it afterward. Accept the snapshot only when both values match. Reconcile the paginated file count and paths with the patch; GitHub's rendered diff and files API have size and count limits, so use exact base/head Git objects when available and report any remaining gap.

Use the returned `headRefOid` to find matching workflow runs:

```bash
gh run list --commit <head-sha> --limit 20 --json conclusion,createdAt,databaseId,displayTitle,event,headSha,name,status,updatedAt,url,workflowName
```

Inspect a failed run only as far as needed to identify the failing job and actionable log section:

```bash
gh run view <run-id> --json conclusion,jobs,status,url,workflowName
gh run view <run-id> --log-failed
```

Use `gh pr checkout <PR>` only when local changes are authorized and the worktree is safe to switch. Confirm local `HEAD` matches `headRefOid` before editing.

## Rebase when GitHub reports conflicts

Rebase only when `mergeable` is `CONFLICTING` or equivalent current provider evidence shows a conflict. Use `baseRefName` as the base and `headRefName` as the PR branch; do not assume conventional names.

Before rebasing, confirm tracked and untracked worktree state and the index are clean, the checked-out branch is the PR head, and its upstream is known. Fetch the base from its actual remote, record the current head SHA and exact fetched base SHA, then rebase the PR branch onto that observed commit:

```bash
git fetch <base-remote> <base-branch>
git rebase <base-sha>
```

Resolve each conflict by intent, stage the resolved paths, and continue with `git rebase --continue`. If a correct resolution is unclear, use `git rebase --abort` and verify the recorded head is restored.

After a successful rebase, run relevant tests and inspect the rewritten range and range-diff:

```bash
git log --oneline <base-remote>/<base-branch>..HEAD
git range-diff <base-sha>..<old-head-sha> <base-sha>..HEAD
```

If push was explicitly authorized, update the PR head branch with a lease:

```bash
git push \
  --force-with-lease=refs/heads/<head-branch>:<old-head-sha> \
  <head-remote> HEAD:refs/heads/<head-branch>
```

Never force-push the base branch. For a fork PR, identify the writable head remote and owner instead of assuming `origin` is the push target. If the explicit lease rejects, refetch and stop rather than retrying with a weaker lease. Refetch PR metadata and require `headRefOid` to match local `HEAD`.

## Collect discussion and review threads

<!-- shared:github-collect-threads -->
Fetch every top-level issue comment:

```bash
gh api 'repos/<owner>/<repo>/issues/<PR>/comments' --paginate
```

Submitted reviews can contain actionable body text without an inline comment. Fetch every review and preserve its `id`, `user.login`, `state`, `body`, `submitted_at`, `commit_id`, and `html_url`:

```bash
gh api 'repos/<owner>/<repo>/pulls/<PR>/reviews' --paginate
```

Inline conversations require GraphQL review threads. Fetch them with their full comment chains and preserve each outer thread `id`:

```bash
gh api graphql --paginate -f query='
  query($owner: String!, $repo: String!, $pr: Int!, $endCursor: String) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100, after: $endCursor) {
          pageInfo { hasNextPage endCursor }
          nodes {
            id
            isResolved
            isOutdated
            path
            line
            comments(first: 100) {
              pageInfo { hasNextPage endCursor }
              nodes { id author { login } body url createdAt }
            }
          }
        }
      }
    }
  }' -F owner='<owner>' -F repo='<repo>' -F pr='<PR>'
```

The outer cursor paginates `reviewThreads` only. For every thread whose nested `comments.pageInfo.hasNextPage` is true, refetch and paginate that thread's complete comment chain separately:

```bash
gh api graphql --paginate -f query='
  query($thread: ID!, $endCursor: String) {
    node(id: $thread) {
      ... on PullRequestReviewThread {
        comments(first: 100, after: $endCursor) {
          pageInfo { hasNextPage endCursor }
          nodes { id author { login } body url createdAt }
        }
      }
    }
  }' -F thread='<thread-id>'
```

Replace the partial chain with the ordered pages from the per-thread query. Do not describe a `first: 100` result as complete without checking its nested `pageInfo`.

Filter for `isResolved == false`. The outer `reviewThreads.nodes[].id`, commonly beginning with `PRRT_`, is the Thread ID. A nested comment ID is not interchangeable with it.
<!-- /shared:github-collect-threads -->

## Reply and resolve

Only perform these mutations when the user authorized them. Put non-trivial Markdown in `reply.md`, then use the exact Thread ID. This is the primary response path for code-line feedback.

<!-- shared:github-thread-reply -->
```bash
gh api graphql \
  -f query='mutation($id: ID!, $body: String!) { addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: $id, body: $body}) { comment { id url } } }' \
  -F id='<thread-id>' \
  -F body=@reply.md
```

Require a returned `.data.addPullRequestReviewThreadReply.comment.url`. Never use `gh pr comment` for an inline review thread; it creates a separate top-level PR comment and breaks the conversation chain.
<!-- /shared:github-thread-reply -->

Resolve only after the reply succeeds and only when the remote PR conclusively contains the fix:

<!-- shared:github-thread-resolve -->
```bash
gh api graphql \
  -f query='mutation($id: ID!) { resolveReviewThread(input: {threadId: $id}) { thread { isResolved } } }' \
  -F id='<thread-id>'
```

Require `.data.resolveReviewThread.thread.isResolved` to be `true`.

If either mutation times out or returns an ambiguous error, refetch the thread before retrying. A reply already visible in the correct thread must not be posted again.
<!-- /shared:github-thread-resolve -->

For an authorized top-level reply:

```bash
gh pr comment <PR> --body-file <file>
```

<!-- shared:github-kody-retrigger -->
After accepted Kody fixes are committed and pushed, an explicitly authorized retrigger is:

```bash
gh pr comment <PR> -b "$(printf '\100kody review')"
```
<!-- /shared:github-kody-retrigger -->

Refetch the PR head, mergeability, checks, workflow runs, and unresolved threads after any push or conversation mutation. Follow only runs for the current `headRefOid`; a green run for an older SHA does not cover the current PR.
