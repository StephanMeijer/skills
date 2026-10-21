---
name: git-commit
description: Create a git commit from the current changes as one Conventional Commit that is GPG-signed, carries a Signed-off-by trailer, and has a short subject line and no body. Analyses the diff to pick type and scope, stages only the files that belong to one change, lets hooks run, and verifies the signature and trailer afterwards. Use when asked to commit, to make a commit, or when "/commit" is mentioned. Do not use to amend or squash an existing pull request's history; use squash-pull-request for that.
license: MIT
---

# Git Commit

One change, one commit, one line. Every commit this skill makes is:

1. **Conventional**: `type(scope): description`, lower-case, imperative, no trailing period, at most 72 characters.
2. **Signed**: GPG-signed with `-S`; a commit that cannot be signed is not made.
3. **Signed off**: carries exactly one `Signed-off-by:` trailer from `-s`.
4. **Subject only**: no body. The subject says what changed; the pull request description says why.

## Set the Authority Boundary

Asking to commit authorizes staging and committing the changes the user is working on. It does not authorize pushing, amending or rewriting published commits, bypassing hooks, changing git configuration, or committing files the user did not touch for this change.

- Never `--no-verify`. If a hook fails for a reason unrelated to the change, fix the environment or report it; do not bypass it.
- Never amend a commit that has been pushed.
- Never commit secrets, credentials, local configuration, build output, or files that are ignored by convention. Say what was left out.

## Read the Change

1. `git status --short` and `git diff` (plus `git diff --staged` when something is already staged). Read the whole diff; the subject must describe what the diff does, not what the user said they did.
2. Read the repository's conventions: `CONTRIBUTING`, `CLAUDE.md`, `AGENTS.md`, `.gitmessage`, `commitlint` or `release-plz` configuration, and `git log --oneline -20`. They decide the allowed types, the scope names, and whether the subject starts lower-case. Follow them over the defaults below.
3. If the diff holds more than one independent concern (a refactor and a behaviour change, a dependency bump and the code that uses it, two unrelated fixes), split it: stage and commit one concern at a time, in dependency order, each with its own subject. Use `git add -p` for hunks that belong to different concerns.

## Choose Type and Scope

| Type | Use when |
|---|---|
| `feat` | behaviour a user can see that did not exist |
| `fix` | behaviour that was wrong is now right |
| `docs` | documentation or comments only |
| `refactor` | structure changes, behaviour does not |
| `perf` | same behaviour, measurably faster or lighter |
| `test` | tests only |
| `build` | build system or dependencies |
| `ci` | CI configuration |
| `chore` | maintenance that fits nothing above |
| `revert` | reverts a commit; subject names it |

Scope is the module, crate, package or area the repository already uses in its history (`fix(http): …`, `docs(docx-reader): …`). Omit it when the change is repository-wide or the history does not use scopes.

A breaking change takes `!` after the scope (`feat(cli)!: …`). It is the one case that adds a line below the subject: a single `BREAKING CHANGE: <what breaks>` footer, because tooling reads it. Keep it to one line.

## Write the Subject

- Imperative mood, present tense: "add", "fix", "drop", "rename". Not "added", "fixes", "adding".
- Say what the change does to the code or behaviour, concretely: `fix(http): classify conversion errors by kind, not by call position`, not `fix(http): improve error handling`.
- No issue numbers, no ticket prefixes, no emoji, no trailing period. Issue links belong in the pull request.
- 72 characters at most, 50 when it fits. If it does not fit, the commit probably holds two changes.

## Commit

```bash
git add <files for this one change>
git commit -S -s -m "<type>(<scope>): <description>"
```

- `-m` with a single argument guarantees there is no body. Do not open an editor; do not pass a second `-m`.
- Let pre-commit and commit-msg hooks run and pass. A hook that rewrites files (formatters) means: re-stage and commit again.
- If signing fails (no key, agent locked, pinentry unavailable), stop and report. Do not fall back to an unsigned commit.

## Verify

```bash
git log -1 --format='%G? %GS%n%s%n%b'
```

- The first line starts with `G` (good signature). `U`, `N`, `E` or `B` mean the commit is not acceptably signed; report it.
- The subject is the one you wrote.
- The body is exactly one `Signed-off-by: Name <email>` line, or that plus one `BREAKING CHANGE:` line. Anything else means a body slipped in: amend with `--amend -S -s -m` only if the commit has not been pushed.
- `git status --short` shows nothing left that belonged to this change.

## Report

- The commit hash and subject, and the signature status.
- What was staged, and anything deliberately left unstaged with the reason.
- Whether the change was split into several commits and why.
- Any hook that ran and what it changed.
- No push was made, unless the user asked for one separately.
