---
name: squash-pull-request
description: Rebase an existing pull request onto its base branch and squash it into exactly one commit that is GPG-signed, carries a Signed-off-by trailer, and has a subject line only, then force-push it with a lease and verify the provider's new head. Works on GitHub and Forgejo, including Codeberg. Use when asked to squash a PR, collapse a PR into one commit, rebase and squash, or clean up a PR's history before merge. Do not use to open a PR (use create-pull-request) or to review one (use pull-request).
license: MIT
---

# Squash Pull Request

Turn a pull request's history into one signed, signed-off commit on top of the current base branch. The result is the PR's net change and nothing else: same files, same content, one commit, one subject line.

## Set the Authority Boundary

Asking to squash a PR authorizes rewriting that PR's branch and force-pushing it with a lease. It does not authorize merging, closing, editing the description, touching the base branch, or discarding commits that exist only locally.

- Never rewrite the base branch.
- Never use plain `--force`; always `--force-with-lease=<branch>:<observed head SHA>`.
- Never reset a local checkout that holds commits the remote does not have; report it instead.

## Establish the Target

1. Treat a PR number or URL as authoritative. Otherwise resolve the PR for the current branch with `gh pr view` or `fj pr view`.
2. Read the exact base branch, head branch, head SHA, author, whether the head is in a fork, mergeability, and the commit list from the provider. Never assume `main`.
3. Fetch both branches from the correct remote. Record the observed head SHA: it is the lease and the recovery point.
4. Check `git worktree list`. If the PR branch is checked out in another worktree, do not try to move it; build the squash on a scratch branch and push that branch to the remote ref instead.
5. Confirm the working tree and index are clean. Do not sweep unrelated local changes into the squash.

## Build the Squash

Do not replay the PR's commits one by one. A PR that contains merge commits or has drifted far from its base conflicts once per replayed commit; the net change conflicts at most once.

```bash
git switch -c squash/<pr> origin/<base>
git merge --squash origin/<head>
```

Resolve any conflict by reading both sides and keeping both intents. Never take `ours` or `theirs` wholesale. If the resolution is not certain, abort, confirm the original head is untouched, and report the conflicting hunks.

Then verify the staged tree is the PR's net change against the base and nothing more:

```bash
diff <(git diff origin/<base> | grep -v '^index ') \
     <(git diff origin/<base>...origin/<head> | grep -v '^index ')
```

An empty diff means equivalence. Differences limited to hunk context (line offsets, a neighbouring line the base changed) are acceptable; content differences are not, unless they are the conflict resolution, which must be explained in the report.

Run the repository's formatter, linter, and the tests for the crates or packages the PR touches, using the toolchain the repository targets. A conflict resolution that builds but is wrong is still wrong.

## Commit

One commit, signed, signed off, subject only:

```bash
git commit -S -s -m "<subject>"
```

- The subject is the PR title unless the user gives one or the repository's commit convention (for example Conventional Commits) requires a different shape.
- No body. The Signed-off-by trailer that `-s` adds is the only line after the subject. Do not add issue references, co-author lines, or attribution unless the user asks.
- Confirm the signature with `git log -1 --format='%G? %GS'`; `G` is a good signature. Confirm the trailer is present and the body is otherwise empty.
- Let commit hooks run. If a hook fails for a reason unrelated to the PR (a toolchain pinned by the environment, a lint the base branch itself fails), fix the environment rather than bypass the hook, and say so in the report.

## Push and Verify

```bash
git push --force-with-lease=<head>:<observed head SHA> origin squash/<pr>:<head>
```

If the lease rejects, the remote moved: refetch, report the new head, and stop. Do not retry with a newer lease without re-reading what changed.

After the push, re-read the PR from the provider and confirm the head SHA is the new commit, the commit count is 1, and mergeability is reported as clean. Provider mergeability can lag a few seconds after a force-push; query again rather than reporting a stale value.

Then put the local branch back in order: delete the stale local branch and recreate it from the remote, or, if it was checked out in another worktree, report the exact `git reset --hard origin/<head>` the user can run there after confirming that worktree holds nothing unpushed.

## Report

- PR link, old head SHA, new head SHA, commit count, signature status, and the exact subject line.
- Whether the squash is byte-equivalent to the PR's net change, and if not, the conflict and how it was resolved.
- Checks run locally and their results.
- Any worktree or local branch left for the user to sync, with the command.
- Anything not done: CI not awaited, hooks worked around, a lease that rejected.
