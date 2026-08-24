# Working a Pull Request as Its Author

Move an existing PR toward merge by turning current CI and reviewer feedback into verified code. Treat local verification as the author's responsibility: never push a change and leave its locally reproducible quality checks for the reviewer to run.

## Rebase a Conflicted PR First

When the provider reports that the PR conflicts with its target branch, rebase before repairing CI or reviewer feedback so subsequent work uses the current integration base.

1. Read the exact base branch and PR head branch from provider metadata. Never assume `main` or `master`.
2. Confirm the current branch is the PR head, tracked and untracked worktree state and the index are clean, its upstream is known, and the base branch is not being rewritten.
3. Fetch the latest base branch from the correct remote and record the pre-rebase PR head SHA as the recovery point.
4. Run a non-interactive rebase of the PR head onto the exact observed base commit.
5. Resolve conflicts by reading both sides and preserving intended behavior. Never select `ours` or `theirs` wholesale.
6. If resolution cannot be completed confidently, run `git rebase --abort`, confirm the original head is restored, and report the blocking conflicts.
7. After a successful rebase, inspect the rewritten branch log and range-diff from the old to new series, account for every changed file, and run the relevant tests.

A rebased PR branch normally requires rewriting its remote history. Do that only when push was explicitly authorized, using a lease that names the exact remote head ref and expected pre-rebase SHA; never use plain `--force` or an implicit lease. If the lease rejects, refetch and report the remote advance instead of retrying destructively. Verify the provider's new head SHA afterward. If push was not authorized, leave the successful local rebase unpushed and report the old and new head SHAs.

## Repair CI/CD Failures

For every failed or cancelled job:

1. Open the job details and isolate the first actionable failure rather than relying on the check title.
2. Reproduce the failing command locally when the repository exposes it.
3. Decide whether the failure is caused by the PR, exposed by the rebase, or external infrastructure such as missing credentials, runner outages, rate limits, or a demonstrably flaky dependency.
4. Fix PR-caused failures within the requested scope. Do not change production code to mask an infrastructure failure or weaken a test to obtain green CI.
5. Re-run the exact failing command, then the broader relevant suite and matching manual usage check.
6. After an authorized push, follow the new pipeline for the pushed head SHA and repeat until it passes or only a proven external blocker remains.

Report external or inaccessible failures with the job URL, observed evidence, and the smallest owner action needed. Do not claim CI is fixed until the provider reports success for the current PR head.

## Execute the Authorized Work

For authorized **needs-work** fixes:

1. Group comments only when one coherent change addresses them together.
2. Reproduce or lock down behavioral defects when practical.
3. Implement the smallest correct change in the repository's established style.
4. Use targeted checks while iterating, then pass the complete local pre-push gate below.
5. Re-read the resulting diff and account for every changed file.
6. Commit and push only when those operations were requested and the pre-push gate passed for the final candidate.
7. After a push, refetch the PR head, CI, mergeability, and open threads.

When implementation is not authorized, propose the smallest credible fix with the path, relevant line or symbol, intended change, expected effect, and validation plan. Include a compact diff when it makes approval easier.

On GitHub, reply using the exact review Thread ID, then resolve that same thread only after the reply succeeds and the remote PR conclusively contains the fix.

## Pass the Local Pre-Push Gate

Before any authorized push, derive the repository's full local check matrix from its guidance, package or task scripts, and CI configuration. Run every applicable, locally reproducible check on the final candidate, including formatting verification, linting, static or type diagnostics, targeted and broader tests, build or packaging checks, and the matching manual usage check. A targeted test alone is not sufficient when the repository provides a broader relevant suite.

Fix every PR-caused failure and rerun the failed check plus any broader check whose inputs changed. Do not push while an applicable local check is failing or has not run. Omit a check only when it is inherently remote, credential-gated, or infrastructure-dependent and no local equivalent exists; record the reason and the evidence instead of asking the reviewer to investigate it.

In the outcome, list the exact local commands and their results. The reviewer should receive verification evidence, not a request to run linting, tests, builds, or manual checks on the author's behalf.
