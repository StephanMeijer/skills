# Forgejo PR review

Use `fj` from a checkout whose selected remote points at the intended Forgejo instance. Codeberg is a Forgejo instance.

## Resolve and inspect the PR

Use the user's PR number when supplied. Otherwise let `fj pr view` resolve the current branch PR and confirm the number before continuing.

```bash
fj pr view <PR>
fj pr view <PR> diff
fj pr view <PR> files
fj pr status <PR>
```

Collect top-level discussion and inline review comments:

```bash
fj pr view <PR> comments
fj pr review <PR> list --comments
```

Use `fj pr checkout <PR>` only when local inspection is necessary, the working tree is safe to switch, and checkout is within the user's request.

## Use the REST API when identifiers are missing

Derive `<host>`, `<owner>`, and `<repo>` from the selected git remote. Forgejo's default API root is `https://<host>/api/v1`.

```bash
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/issues/<PR>/comments?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>/reviews?limit=50&page=1'
curl -fsS 'https://<host>/api/v1/repos/<owner>/<repo>/pulls/<PR>/reviews/<review-id>/comments?limit=50&page=1'
```

Paginate until a page contains fewer than the requested limit. For private repositories, use an authenticated API client or a Forgejo token without printing or logging the token.

An unresolved review comment has a null `resolver`. Preserve its `id`, `pull_request_review_id`, `path`, `position`, and `html_url` for triage.

## Reply limitations

Forgejo's current REST API exposes review comments and resolver state, but no reply-in-thread or resolve-conversation operation. The review-comment creation endpoint creates a review comment; it does not attach a reply to an existing conversation.

For a true threaded response, open the inline comment's `html_url`, reply in the Forgejo web UI, and resolve there only when explicitly authorized and conclusively addressed.

Do not present this as an inline reply:

```bash
fj pr comment <PR> '<message>'
```

That command creates a top-level PR comment. Use it only for explicitly authorized top-level discussion. Top-level comments have no resolved state.

Check `fj pr review --help` before attempting an overall approval or change request. If the installed client exposes only review listing, use the Forgejo web UI rather than guessing an API mutation.

After accepted Kody fixes are committed and pushed, an explicitly authorized top-level retrigger is:

```bash
fj pr comment <PR> "$(printf '\100kody review')"
```
