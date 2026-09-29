# Repository Profile Template

Create `$state_dir/PROFILE.md` on the first tick. Derive every entry from checked-in configuration (CI workflows, task runners, repository guidance), not from memory. Sub-agents read it, so keep it short and literal.

```markdown
# PR steward profile: <owner>/<repo>

## Branches
- Base branch: <main>
- Merge style: <merge commits | squash | rebase>. <Any rule this implies, e.g. every commit needs its own breaking-change marker.>
- Commit convention: <e.g. Conventional Commits, as enforced by release tooling>

## Local gate (run on the final candidate before every push)
- <exact command 1, e.g. cargo fmt --all -- --check>
- <exact command 2>
- <exact command 3>
- Docs-only changes: <which checks still apply>

## Build environment for sub-agents
- <env vars, e.g. a shared target directory so parallel agents reuse one build cache>
- Toolchain: <pinned version, and why the shell default is not enough>
- Disk floor: stop when free space on <mount> is below <N> GB. Never clean shared caches.

## CI
- Required checks: <names>
- AI review lanes that must finish before a PR is ready: <workflow and lane names>
- Known queue behaviour: <e.g. concurrency groups without cancel-in-progress; long queues are not failures>

## Known flaky tests
- <test name>: <failure signature>. Rerun once; if it passes, note it.

## Merge-order hazards
- <PR or change>: <what other PRs must do while it is open or after it lands>

## Excluded
- <PRs the user said to leave alone>
```
