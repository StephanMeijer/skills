# Forgejo pull requests

Use `fj` from a checkout whose selected remote points at the intended Forgejo instance. Codeberg is a Forgejo instance.

## Resolve and inspect the PR

Use an explicit PR number when supplied. Otherwise let `fj pr view` resolve the current branch PR and confirm the number before continuing.

```bash
fj pr view <PR>
fj pr view <PR> diff
fj pr view <PR> files
fj pr status <PR>
```

Read the PR's head SHA before collecting other surfaces and again afterward. Accept the snapshot only when the head is unchanged. Keep the terminal `Link` state and any declared total with each paginated collection; a short page is not proof of completion.

Collect top-level discussion and inline review comments:

```bash
fj pr view <PR> comments
fj pr review <PR> list --comments
```

Use `fj pr checkout <PR>` only when local inspection or authorized local changes require it and the worktree is safe to switch. Compare local `HEAD` with the provider's PR head before editing.

## Use the REST API for complete identifiers and CI

Derive `<host>`, `<owner>`, and `<repo>` from the selected git remote. Forgejo's default API root is `https://<host>/api/v1`.

```bash
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/issues/<PR>/comments?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>/reviews?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>/reviews/<review-id>/comments?limit=50&page=1'
```

Paginate by following the response's `Link` header with `rel="next"` until no next link remains. Do not stop because a page contains fewer than the requested limit: a Forgejo instance can clamp `limit=50` to its lower `max_response_items` setting. Query `/api/v1/settings/api` when the client's effective page size must be diagnosed. For private repositories, use an authenticated API client or a Forgejo token without printing or logging the token.

An unresolved review comment has a null `resolver`. Preserve its `id`, `pull_request_review_id`, `path`, `position`, and `html_url` for triage.

Reconcile the provider's changed-file inventory with the patch. When the provider truncates either surface, use exact base/head Git objects when available without altering the user's worktree and report any binary, LFS, submodule, fork-ref, or oversized-file gap.

Read the PR response's head SHA, then inspect commit statuses and matching action runs when available:

```bash
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/commits/<head-sha>/statuses?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/actions/runs?limit=50&page=1'
```

Filter action runs by the exact head SHA. Treat an unavailable actions endpoint as unavailable evidence, not as a passing pipeline.

## Rebase when Forgejo reports conflicts

Use the PR metadata's base ref and head ref; never assume `main` or `master`. Rebase only when the provider reports a merge conflict.

Before rebasing, confirm tracked and untracked worktree state and the index are clean, the checked-out branch is the PR head, and its upstream is known. Fetch the exact base branch, record the current PR head SHA and fetched base SHA, and rebase onto that observed commit:

```bash
git fetch <base-remote> <base-branch>
git rebase <base-sha>
```

Resolve each conflict by intent and continue with `git rebase --continue`. If a correct resolution is unclear, run `git rebase --abort` and verify the recorded head is restored. Then run relevant tests and inspect the rewritten branch range and `git range-diff <base-sha>..<old-head-sha> <base-sha>..HEAD`.

If push was explicitly authorized, update the PR head with a lease:

```bash
git push \
  --force-with-lease=refs/heads/<head-branch>:<old-head-sha> \
  <head-remote> HEAD:refs/heads/<head-branch>
```

Never force-push the base branch. For a fork PR, identify the writable head remote rather than assuming `origin`. If the explicit lease rejects, refetch and stop rather than retrying with a weaker lease. Refetch the PR and verify its head SHA matches local `HEAD` after the push.

## Submit fresh findings as inline comments

Only submit after explicit user authorization. Forgejo accepts a review with an array of inline comments at `POST /repos/<owner>/<repo>/pulls/<PR>/reviews`.

Create `review.json` with the reviewed head SHA and one comment per defect:

```json
{
  "commit_id": "<head-sha>",
  "body": "Review summary and any unanchorable findings.",
  "event": "COMMENT",
  "comments": [
    {
      "path": "src/example.ts",
      "new_position": 42,
      "body": "Explain the impact, triggering scenario, and smallest credible fix."
    }
  ]
}
```

Use `new_position` for a line on the new side of the diff and `old_position` for a deleted line on the old side. Set only the applicable side. Anchor only to lines in the reviewed diff; keep cross-cutting or unanchorable findings in the review body. Use `COMMENT` unless the user explicitly requested `APPROVED` or `REQUEST_CHANGES`.

When the installed `fj` exposes raw API access, submit the review with:

```bash
fj api '/repos/<owner>/<repo>/pulls/<PR>/reviews' \
  -X POST \
  --input review.json
```

Otherwise use an already configured authenticated API client against the same instance endpoint. If none is available, open the PR's changed-files view, add each finding on its intended line with the web UI, and submit the review there. For API submission, require the returned review `id`, `html_url`, `commit_id`, and expected `comments_count`, then list `/repos/<owner>/<repo>/pulls/<PR>/reviews/<review-id>/comments` and require an `html_url` for every intended inline finding. For web submission, reopen the submitted review and verify every intended inline comment in place.

If Forgejo rejects an anchor, refetch the diff and correct the old/new line selection. Do not silently convert the finding into a top-level PR comment.

## Reply and resolve limitations

Forgejo's current REST API exposes review comments and resolver state, but no reply-in-thread or resolve-conversation operation. The review-comment creation endpoint creates a review comment; it does not attach a reply to an existing conversation.

For a true threaded response, open the inline comment's `html_url`, reply in the Forgejo web UI, and resolve there only when explicitly authorized. As a reviewer, resolve when the thread is conclusively addressed; as the PR author, resolve only when the remote PR conclusively contains the fix.

Do not present this as an inline reply:

```bash
fj pr comment <PR> '<message>'
```

That command creates a top-level PR comment. Use it only for explicitly authorized top-level discussion. Top-level comments have no resolved state.

Check `fj pr review --help` before attempting an overall approval or change request. If the installed client exposes only review listing, use the Forgejo web UI rather than guessing an API mutation.

Kodus documents `@kody start-review` as the manual review trigger. A plain trigger is correct here because the accepted fixes are new commits; reserve `@kody review --force` for a pass Kody would otherwise skip. Confirm the command against the Kody version actually installed on the repository before relying on it.

After accepted Kody fixes are committed and pushed, an explicitly authorized top-level retrigger is:

```bash
fj pr comment <PR> "$(printf '\100kody start-review')"
```

`printf` emits the leading `@` at run time so that quoting this command in a report, commit message, or PR comment cannot mention the bot by accident.

Refetch PR state, mergeability, CI, comments, and unresolved review items after any push or remote mutation. Follow only CI attached to the current PR head SHA. Include every thread URL that still requires web-UI action in the final report.
