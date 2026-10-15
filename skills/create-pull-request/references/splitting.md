# Splitting Work into Small PRs

A small PR gets reviewed sooner, reviewed better, and reverted cleanly. Split by concern, not by file count or line count.

## Where to Cut

Separate these into their own PRs, in roughly this merge order:

1. **Preparatory refactors** that change no behavior: renames, moves, extracting a function, introducing a type. Reviewers can approve these quickly because tests should not change.
2. **Mechanical changes**: formatting, codemods, dependency bumps, generated code. Reviewers skim these; mixing them with logic hides the logic.
3. **Infrastructure the feature needs**: a migration, a new endpoint, a config option, behind a flag or unused until the next PR.
4. **The behavior change itself**, with its tests.
5. **Follow-ups**: cleanup, removing the old path, documentation.

Independent features or fixes always get separate PRs, even when they were written together.

## When Not to Split

- The pieces cannot build or pass tests alone, and no flag or seam makes them do so.
- Splitting would leave the product in a broken or half-migrated state between merges.
- The change is one indivisible concern, such as a single algorithm rewrite. Say so in the description and guide the reviewer through the diff instead.

Never split a test away from the code it tests.

## Independent or Stacked

- **Independent PRs** each target the default branch. Use this whenever the pieces do not depend on each other; they can be reviewed and merged in any order.
- **Stacked PRs** form a chain: the first targets the default branch, each next one targets the previous PR's branch. Use this only when a later PR needs an earlier one. Keep stacks short, ideally no more than three.

In a stack, every description lists the whole chain:

```markdown
## Stack

1. #101 Extract session store interface
2. **#102 Add Redis session store** (this PR)
3. #103 Switch production to Redis sessions
```

After a lower PR merges, the next PR's base must move to the default branch. Mention this in the report; doing the retarget and rebase later is work for the `pull-request` skill.

## Building the Branches

Split local commits without touching anything already pushed:

1. Record the current branch head as a recovery point.
2. For each planned PR, create a branch from its base and bring over its changes: `git cherry-pick` whole commits when the commits already match the split, or check out whole files with `git checkout <source> -- <paths>`, or apply part of the diff with `git diff <base> <source> -- <paths> | git apply`. Interactive commands such as `git add -p` or `git rebase -i` are not available.
3. Commit with messages that follow the repository's convention.
4. Verify that each branch builds and passes tests on its own, and that the final branch of a stack matches the original combined change: `git diff <original-head> <last-branch>` should be empty, or contain only intended differences.
