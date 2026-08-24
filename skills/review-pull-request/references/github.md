# GitHub PR review

Use authenticated `gh` commands from a checkout whose `origin` points at GitHub.

## Resolve and inspect the PR

Use the user's PR number when supplied. Otherwise resolve the current branch PR:

```bash
gh pr view --json number -q .number
```

Collect summary data and inspect the patch without changing branches:

```bash
gh pr view <PR> --json number,title,state,isDraft,url,author,baseRefName,headRefName,headRefOid,body,mergeable,reviewDecision
gh pr diff <PR>
gh pr checks <PR> --json bucket,completedAt,description,event,link,name,startedAt,state,workflow
gh api 'repos/<owner>/<repo>/pulls/<PR>/files?per_page=100' --paginate
```

Record `headRefOid` before collecting other surfaces and refetch it afterward. Accept the snapshot only when both values match. Reconcile the paginated file count and paths with the patch; GitHub's rendered diff and files API have size and count limits, so use exact base/head Git objects when available and report any remaining gap.

Use the PR's `headRefOid` to find matching workflow runs:

```bash
gh run list --commit <head-sha> --limit 20 --json conclusion,createdAt,databaseId,displayTitle,event,headSha,name,status,updatedAt,url,workflowName
```

Use `gh pr checkout <PR>` only when local inspection is necessary, the working tree is safe to switch, and checkout is within the user's request.

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

## Submit fresh findings as inline comments

Only submit after explicit user authorization. Prefer one review containing every fresh inline finding so the findings share one disposition and do not produce a burst of separate notifications.

Create `review.json` with the reviewed head SHA and one comment per defect:

```json
{
  "commit_id": "<head-sha>",
  "body": "Review summary and any unanchorable findings.",
  "event": "COMMENT",
  "comments": [
    {
      "path": "src/example.ts",
      "line": 42,
      "side": "RIGHT",
      "body": "Explain the impact, triggering scenario, and smallest credible fix."
    }
  ]
}
```

Use `RIGHT` for an added or context line and `LEFT` for a deleted line. Use the blob line number visible in the PR diff, not the diff offset. For a multi-line finding, add `start_line` and `start_side`. Anchor only to lines in the reviewed diff; keep cross-cutting or unanchorable findings in the review body.

Use `COMMENT` unless the user explicitly requested `APPROVE` or `REQUEST_CHANGES`. Submit the review through the API because `gh pr review` cannot attach the `comments` array:

```bash
gh api --method POST \
  'repos/<owner>/<repo>/pulls/<PR>/reviews' \
  --input review.json
```

Require the response's `id`, `html_url`, `state`, and `commit_id`. Then fetch the created comments and require a returned `html_url` for every intended inline finding:

```bash
gh api 'repos/<owner>/<repo>/pulls/<PR>/reviews/<review-id>/comments'
```

If GitHub rejects an anchor, refetch the diff and correct the line or side. Do not silently convert the finding into a top-level PR comment.

## Reply inside an existing thread

Only reply after explicit user authorization. Put non-trivial Markdown in `reply.md`, then use the exact Thread ID. This is the primary response path for code-line feedback.

<!-- shared:github-thread-reply -->
```bash
gh api graphql \
  -f query='mutation($id: ID!, $body: String!) { addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: $id, body: $body}) { comment { id url } } }' \
  -F id='<thread-id>' \
  -F body=@reply.md
```

Require a returned `.data.addPullRequestReviewThreadReply.comment.url`. Never use `gh pr comment` for an inline review thread; it creates a separate top-level PR comment and breaks the conversation chain.
<!-- /shared:github-thread-reply -->

When the thread is conclusively addressed and resolution was explicitly authorized, resolve it only after posting the reply:

<!-- shared:github-thread-resolve -->
```bash
gh api graphql \
  -f query='mutation($id: ID!) { resolveReviewThread(input: {threadId: $id}) { thread { isResolved } } }' \
  -F id='<thread-id>'
```

Require `.data.resolveReviewThread.thread.isResolved` to be `true`.

If either mutation times out or returns an ambiguous error, refetch the thread before retrying. A reply already visible in the correct thread must not be posted again.
<!-- /shared:github-thread-resolve -->

## Other remote actions

Top-level issue comments have no resolved state. Use this only for explicitly authorized top-level discussion:

```bash
gh pr comment <PR> --body-file <file>
```

When there are no inline findings, submit an explicitly authorized overall-only review with exactly one disposition:

```bash
gh pr review <PR> --comment --body-file <file>
gh pr review <PR> --approve --body-file <file>
gh pr review <PR> --request-changes --body-file <file>
```

<!-- shared:github-kody-retrigger -->
After accepted Kody fixes are committed and pushed, an explicitly authorized retrigger is:

```bash
gh pr comment <PR> -b "$(printf '\100kody review')"
```
<!-- /shared:github-kody-retrigger -->
