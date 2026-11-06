---
name: squash-pull-request
description: Rebase a pull request onto its base branch and squash it into exactly one GPG-signed, signed-off, subject-only commit, then force-push it with a lease and verify the provider's new head. Works on GitHub and Forgejo, including Codeberg. Use when asked to squash a PR, collapse a PR into one commit, rebase and squash, or clean up a PR's history before merge. Do not use to open a PR (use create-pull-request) or to review one (use pull-request).
license: MIT
---

# Squash Pull Request

Turn a pull request's history into one signed, signed-off commit on top of the current base branch: same files, same content, one commit, one subject line.

## Set the Authority Boundary

Asking to squash a PR authorizes rewriting that PR's branch and force-pushing it with a lease. It does not authorize merging, closing, editing the description, rewriting the base branch, or discarding commits that exist only locally.

- Never use plain `--force`; always `--force-with-lease=<branch>:<observed head SHA>`.
- Never reset a local checkout that holds commits the remote does not have; report it instead.

## Establish the Target

1. A PR number or URL is authoritative; otherwise resolve the PR for the current branch with `gh pr view` or `fj pr view`.
2. Read the base branch, head branch, head SHA, author, whether the head is in a fork, mergeability, and the commit list from the provider. Never assume `main`.
3. Fetch both branches from the correct remote. Record the observed head SHA: it is the lease and the recovery point.
4. Check `git worktree list` and note whether the PR branch is checked out in another worktree; the cleanup after the push depends on it.
5. Confirm the working tree and index are clean so nothing unrelated is swept into the squash.

## Build the Squash

Do not replay the PR's commits one by one; squash the net change, which conflicts at most once.

```bash
git switch -c squash/<pr> origin/<base>
git merge --squash origin/<head>
```

Resolve a conflict by reading both sides and keeping both intents; never take `ours` or `theirs` wholesale. If the resolution is not certain, abort, confirm the original head is untouched, and report the conflicting hunks.

Then verify the staged tree is exactly the PR's net change:

```bash
diff <(git diff origin/<base> | grep -v '^index ') \
     <(git diff origin/<base>...origin/<head> | grep -v '^index ')
```

An empty diff means equivalence. Hunk-context differences (line offsets, a neighbouring line the base changed) are acceptable; content differences are not, unless they are the conflict resolution, which the report must explain.

Run the repository's formatter, linter, and the tests for the crates or packages the PR touches, with the toolchain the repository targets.

## Commit

One commit, signed, signed off, subject only:

```bash
git commit -S -s -m "<subject>"                                  # or, for a breaking change:
git commit -S -s -m "<subject>" -m "BREAKING CHANGE: <what breaks>"
```

- Subject per the `git-commit` skill: Conventional by default (`type(scope): description`, imperative, at most 72 characters), describing the PR's net change. The PR title is input, not the subject, unless the user gives a subject or the repository's convention differs.
- No body, with one exception: when a squashed commit carries `!` after its scope or a `BREAKING CHANGE:` footer, keep `!` in the subject and carry exactly one `BREAKING CHANGE:` line as the only body line, because release tooling reads it. No issue references, co-author lines, or attribution unless the user asks.
- Confirm the signature with `git log -1 --format='%G? %GS'`; `G` is a good signature. Confirm the body is exactly the Signed-off-by trailer from `-s`, or that plus the one `BREAKING CHANGE:` line.
- Let commit hooks run. If a hook fails for a reason unrelated to the PR (a toolchain pinned by the environment, a lint the base branch itself fails), fix the environment rather than bypass the hook, and say so in the report.

## Push and Verify

```bash
git push --force-with-lease=<head>:<observed head SHA> origin squash/<pr>:<head>
```

If the lease rejects, the remote moved: refetch, report the new head, and stop. Do not retry with a newer lease without re-reading what changed.

Re-read the PR from the provider and confirm the head SHA is the new commit, the commit count is 1, and mergeability is clean. Mergeability can lag a few seconds after a force-push; query again.

Then delete the stale local branch and recreate it from the remote, or, if it was checked out in another worktree, report the exact `git reset --hard origin/<head>` the user can run there once that worktree holds nothing unpushed.

## Stacked PRs

When the PR is part of a GitHub stack (`gh stack view`, extension `github/gh-stack`), squashing rewrites the branch the PRs above it are built on. Say so before squashing. After the push, rebase them with `gh stack rebase` only if the user authorized rewriting those branches too; otherwise list them in the report as needing a rebase. If the extension is missing, ask the user to install it (`gh extension install github/gh-stack`) rather than installing it yourself.

## Report

- PR link, old and new head SHA, commit count, signature status, and the exact subject line.
- Whether the squash is byte-equivalent to the PR's net change, and if not, the conflict and how it was resolved.
- Checks run locally and their results.
- Any worktree or local branch left for the user to sync, with the command.
- Anything not done: CI not awaited, hooks worked around, a lease that rejected.
