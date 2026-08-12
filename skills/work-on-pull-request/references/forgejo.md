# Forgejo PR author workflow

Use `fj` from a checkout whose selected remote points at the intended Forgejo instance. Codeberg is a Forgejo instance.

## Resolve and inspect the PR

Use an explicit PR number when supplied. Otherwise let `fj pr view` resolve the current branch PR and confirm its number before continuing.

```bash
fj pr view <PR>
fj pr view <PR> diff
fj pr view <PR> files
fj pr status <PR>
```

Use `fj pr checkout <PR>` only when local changes are authorized and the worktree is safe to switch. Compare local `HEAD` with the provider's PR head before editing.

Collect top-level discussion and inline review comments:

```bash
fj pr view <PR> comments
fj pr review <PR> list --comments
```

## Rebase when Forgejo reports conflicts

Use the PR metadata's base ref and head ref; never assume `main` or `master`. Rebase only when the provider reports a merge conflict.

Before rebasing, confirm the worktree and index are clean, the checked-out branch is the PR head, and its upstream is known. Fetch the exact base branch, record the current PR head SHA, and rebase:

```bash
git fetch <base-remote> <base-branch>
git rebase <base-remote>/<base-branch>
```

Resolve each conflict by intent and continue with `git rebase --continue`. If a correct resolution is unclear, run `git rebase --abort` and verify the recorded head is restored. Then run relevant tests and inspect the rewritten branch range.

If push was explicitly authorized, update the PR head with a lease:

```bash
git push --force-with-lease <head-remote> HEAD:<head-branch>
```

Never force-push the base branch. For a fork PR, identify the writable head remote rather than assuming `origin`. Refetch the PR and verify its head SHA matches local `HEAD` after the push.

## Use the REST API for complete identifiers and CI

Derive `<host>`, `<owner>`, and `<repo>` from the selected remote. Forgejo's default API root is `https://<host>/api/v1`.

```bash
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/issues/<PR>/comments?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>/reviews?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>/reviews/<review-id>/comments?limit=50&page=1'
```

Paginate by following the response's `Link` header with `rel="next"` until no next link remains. Do not stop because a page contains fewer than the requested limit: a Forgejo instance can clamp `limit=50` to its lower `max_response_items` setting. Query `/api/v1/settings/api` when the client's effective page size must be diagnosed. An unresolved review comment has a null `resolver`; preserve its `id`, `pull_request_review_id`, `path`, `position`, and `html_url`.

Read the PR response's head SHA, then inspect commit statuses and matching action runs when available:

```bash
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/commits/<head-sha>/statuses?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/actions/runs?limit=50&page=1'
```

Filter action runs by the exact head SHA. Treat an unavailable actions endpoint as unavailable evidence, not as a passing pipeline. Use an authenticated client for private repositories without printing or logging its token.

## Reply and resolve limitations

Forgejo's REST API exposes review comments and resolver state but does not currently provide reply-in-thread or resolve-conversation operations. Use the inline comment's `html_url` in the Forgejo web UI for an authorized true threaded reply or resolution.

Do not present this as an inline reply:

```bash
fj pr comment <PR> '<message>'
```

That command creates a top-level PR comment. Use it only for explicitly authorized top-level discussion. Top-level comments have no resolved state.

Check `fj pr review --help` before attempting an authorized overall review action. If the installed client exposes only review listing, use the web UI instead of guessing an API mutation.

After accepted Kody fixes are pushed, an explicitly authorized top-level retrigger is:

```bash
fj pr comment <PR> "$(printf '\100kody review')"
```

Refetch PR state, mergeability, CI, comments, and unresolved review items after any push or remote mutation. Follow only CI attached to the current PR head SHA. Include every thread URL that still requires web-UI action in the final report.
