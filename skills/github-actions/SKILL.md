---
name: github-actions
description: Write, review, harden, and debug GitHub Actions workflows and in-repository actions. Pins every action to a commit SHA in a lockfile with the gh actions-lock extension, grants least-privilege token permissions, closes script-injection and pull_request_target holes, keeps runs fast with caching, concurrency, and timeouts, and debugs failing runs from their logs with the gh CLI. Use when asked to add, change, review, secure, speed up, or fix a GitHub Actions workflow, CI pipeline, release or deploy job, composite or JavaScript action, Dependabot setup for actions, or a failing or flaky run. Do not use for other CI systems such as Forgejo Actions, GitLab CI, or Jenkins.
license: MIT
---

# GitHub Actions

Make workflows that do what the repository needs, with the least privilege and the fewest moving parts, and prove it with a real run. A workflow is code that runs with the repository's token and secrets: treat every change as a security change.

## Establish the Contract

1. Read every file under `.github/workflows/`, in-repository actions (`action.yml` anywhere), `.github/dependabot.yml`, `.github/workflows/actions.lock`, and the repository's guidance (`CONTRIBUTING`, `CLAUDE.md`, `AGENTS.md`).
2. Derive what the workflow must run from the repository's own commands: task runners, package scripts, Makefiles, documented build and test steps. Never invent a command that does not run locally.
3. For an existing workflow, read its recent runs (`gh run list --workflow <file>`) and note durations, failures, and which checks branch protection requires. A required check that is renamed or removed blocks every PR.
4. Keep triggers, job names, and check names stable unless the user asks for a change; rulesets and branch protection refer to them by name.

## Apply the Security Policy

Read [references/security.md](references/security.md) before writing or reviewing any workflow. In short:

- Pin every action to a full commit SHA, recorded in `.github/workflows/actions.lock` with GitHub's `gh actions-lock` extension (below).
- Start from `permissions: {}` or `contents: read` at the top level and grant more per job only where a step needs it.
- Never interpolate untrusted `${{ }}` expressions into `run:` scripts; pass them through `env:`.
- Never check out or run pull request code in a `pull_request_target` or `workflow_run` job that has secrets or a write token.
- Prefer OIDC (`id-token: write`) over long-lived cloud credentials, and environments with required reviewers for deploys.

## Pin Actions with the Lockfile

GitHub's `gh actions-lock` extension pins every direct and transitive action dependency to an immutable commit SHA in `.github/workflows/actions.lock`, so action changes show up in pull request diffs and moved or tampered tags are caught before they run. Check `gh extension list` for `github/gh-actions-lock`; if it is missing, ask the user to install it (`gh extension install github/gh-actions-lock`) rather than installing it yourself, since it runs code from outside the repository with the user's token.

- `gh actions-lock` scans all workflows, pins what it can, and writes the lockfile. Commit the lockfile with the workflow change that needed it.
- `gh actions-lock --verify` is a read-only check that fails on an incomplete lockfile or a pin the runner would reject. Run it before every push that touches workflows, and add it to CI when the repository uses a lockfile.
- `gh actions-lock --verify-local` is an offline coverage check, fit for a pre-commit hook. It does not prove the entries are trustworthy.
- A partial version ref (`@v4`) is narrowed to the highest full semver tag when first pinned; keep that unless the user wants otherwise (`--no-narrow`).
- A **moved ref** or **unreachable pin** is a supply-chain signal, not noise. Inspect what the tag or branch now points to and why before running `--accept-moved` or `--relock`, and report what moved.
- Same-repository `uses: ./path` actions can be migrated to the inherently pinned `$/` form with `--migrate-local-actions`; do it only when asked.
- A repository without a lockfile is not onboarded unasked. Suggest it in the report, with the number of unpinned actions found.

Without the extension, pin `uses:` to a full 40-character SHA with the version as a trailing comment (`actions/checkout@<sha> # v4.2.2`), resolved from the action's own repository with `gh api repos/<owner>/<repo>/git/ref/tags/<tag>`, and dereference annotated tags to their commit.

Keep actions updated with Dependabot (`package-ecosystem: github-actions`). After a bump, rerun `gh actions-lock` so the lockfile follows.

## Write Lean Workflows

- Trigger precisely: `pull_request` for checks on PRs, `push` to the default branch for post-merge work, `paths` or `paths-ignore` filters when a job only matters for part of the tree. Avoid running the same check on both `push` and `pull_request` for the same commit unless the repository relies on it.
- Add `concurrency` keyed on the workflow and ref, with `cancel-in-progress: true` for PR checks, never for deploys or releases.
- Set `timeout-minutes` on every job; the default is six hours.
- Use the setup actions' built-in caching (`cache:` on `setup-node`, `setup-python`, `setup-go`) or `actions/cache` keyed on lockfile hashes. Never cache secrets or build outputs that embed them.
- Use a `matrix` only for dimensions the project really supports; `fail-fast: false` when each cell's result matters on its own.
- Extract repeated job sequences into a reusable workflow (`workflow_call`) or composite action rather than copying them.
- Keep logic in repository scripts the developer can run locally; the workflow should call them, not reimplement them in YAML.

## Verify Locally, Then Remotely

1. Run `actionlint` on every changed workflow; it also runs ShellCheck on `run:` scripts when ShellCheck is installed. Run `zizmor` for security findings when installed. Report which linters were unavailable.
2. Run `gh actions-lock --verify` when the repository has a lockfile.
3. Run the commands the workflow calls locally, with the same versions the workflow pins where practical.
4. Push the change on a branch or PR and watch the real run: `gh run watch <run-id>`, or `gh run list --branch <branch>` then `gh run view <run-id>`. A workflow is not verified until GitHub ran it green. For `workflow_dispatch`, start it with `gh workflow run <file> --ref <branch>`.
5. For triggers that cannot run from a branch (a `release`, a schedule, `pull_request_target`), say exactly what was not exercised.

## Debug Failing Runs

Read [references/debugging.md](references/debugging.md) for the full procedure. Start with `gh run view <run-id> --log-failed`, find the first actionable error rather than the last red line, reproduce the step's command locally, and decide whether the cause is the change, the base branch, or infrastructure (runner outage, rate limit, expired secret, a moved action ref). Rerun only failed jobs (`gh run rerun <run-id> --failed`) when the cause is shown to be transient, and never weaken a check to make it pass.

## Report

List the workflows and actions changed, the triggers, permissions granted per job and why, actions pinned or the lockfile change, linters run and their results (and which were unavailable), the run URL that proves the change green, and anything that could not be exercised. Name configuration and secrets by name only; never print their values.
