---
name: create-pull-request
description: Open new pull requests on GitHub and Forgejo, including Codeberg, that are small, verified, and easy to review. Splits large changes into several focused PRs (stacked when they depend on each other), writes a description with an ELI5 summary for every PR, and captures and uploads before and after screenshots whenever the change is visual. Use when asked to open, create, raise, or submit a PR or merge request, to turn local commits or a branch into PRs, or to split a large change into smaller PRs. Do not use to review, fix, or rebase an existing PR; use pull-request for that.
license: MIT
---

# Create Pull Request

Turn local work into pull requests a reviewer can understand in minutes. Three rules apply to every PR:

1. **Small.** Several focused PRs over one large one. Each PR does one thing and passes its checks alone.
2. **ELI5.** Every description opens with a plain-language explanation anyone on the team could follow.
3. **Screenshots when visual.** Any change a user can see ships with screenshots in the description.

## Set the Authority Boundary

Asking to create a PR authorizes pushing the branches and opening the PRs. It does not authorize merging, approving, closing other PRs, force-pushing a branch someone else is working on, or requesting reviewers not named by the repository's conventions or the user.

- Never push to the default branch or rewrite its history.
- Rewriting local, unpushed commits to split them is allowed; rewriting already-pushed commits needs the user's approval.
- When the user explicitly asked for a single PR and the change is large, open one PR and say in the report how it could have been split.

## Establish the Ground

1. Read `git remote get-url origin` to tell GitHub (`gh`) from Forgejo or Codeberg (`fj` and the REST API).
2. Take the target branch from the provider's default branch or the user's instruction. Never assume `main`.
3. Inspect `git status`, the current branch, its upstream, and the commits and uncommitted changes relative to the target. Do not sweep unrelated local changes into a PR; ask about them.
4. Read the repository's guidance: `CONTRIBUTING`, `CLAUDE.md`, `AGENTS.md`, the PR template (`.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`, `.forgejo/`, `.gitea/`), commit message conventions, branch naming, and labels.
5. Search open PRs and issues for overlapping work and for the issues this work resolves.

If the change may fix an undisclosed vulnerability, read the repository's `SECURITY.md` and [references/security-fixes.md](references/security-fixes.md) before inspecting public trackers, naming a public branch, pushing, or opening a PR. Never make private exploit details public as a side effect of PR creation.

## Plan Small PRs

Read [references/splitting.md](references/splitting.md) and decide the split before writing any description.

Split when the change contains more than one independent concern: a refactor and a behavior change, a mechanical rename and new logic, a dependency bump and the code that uses it, or features that could ship separately. Split a PR above roughly 400 changed lines, excluding generated files, lockfiles, and snapshots, unless it is one indivisible concern.

State the plan in one short list (PR title, contents, base) and proceed. The titles in the plan are already the plain-English PR titles, never commit subjects. Each planned PR must build, pass tests, and leave the product working when merged alone, in order.

## Verify Before Opening

For each PR branch, run the repository's documented build, lint, typecheck, and test commands locally. Do not open a PR whose locally reproducible checks fail; fix the failure or report it. Open as a draft only when the user asks, the repository's conventions say so, or a known failure remains that the user accepted.

When the repository has `.github/workflows/actions.lock`, or the change adds or edits anything under `.github/workflows/` or an in-repo action, run `gh actions-lock` (GitHub's `github/gh-actions-lock` extension; if missing, ask the user to run `gh extension install github/gh-actions-lock`). It pins every action to a commit SHA and updates the lockfile; the lockfile change belongs in the same PR. Verify with `gh actions-lock --verify`, which writes nothing and fails on an incomplete lockfile. Do not onboard a repository without a lockfile unasked; suggest it in the report.

A private security fix runs the complete locally reproducible gate; hosted CI and integrations may be unavailable in the temporary private fork, so record anything only the maintainer can verify after merging or publishing.

## Capture Screenshots for Visual Changes

A change is visual when anything a user sees could look different: UI components, styles, layout, copy, icons, images, generated documents, charts, emails, or CLI output formatting. When in doubt, treat it as visual.

Follow [references/screenshots.md](references/screenshots.md): capture before and after screenshots of every affected screen or state and upload them to the PR description with `gh pr create --attach` or `gh pr edit --attach`. Never commit screenshots to the PR branch.

## Write the Description

Use [references/description-template.md](references/description-template.md). When the repository has its own PR template, fill it in and add the ELI5 and Screenshots sections rather than replacing it.

- The **ELI5** comes first: two to four short sentences in everyday words, no jargon, no file or function names, saying what changes and why it matters to the person using or running the product.
- The title says what changes for the user or operator, in plain words, for someone scanning a PR list. A PR title is never a commit message: no copied commit subject and no Conventional Commit syntax (`feat(scope): …`), in the plan, in `gh pr create`, or in a later edit. For example, commit `fix(auth): accept the bearer scheme case-insensitively` becomes the PR `Accept lowercase "bearer" in the Authorization header`. Follow the house style of recent merged PR titles, such as a `Fix:` prefix.
- Link issues with closing keywords (`Closes #123`) only when the PR fully resolves them; otherwise reference them.
- For stacked PRs, list the whole stack with links in every PR and mark which one this is.
- Write the description to a file and pass it with `--body-file`, so quoting and newlines survive.

## Open the PRs

Push each branch with an upstream, then open the PR against its base.

```bash
git push -u origin <branch>
gh pr create --base <base> --head <branch> --title "<title>" --body-file <file>
```

For Forgejo, use `fj pr create` (check `fj pr create --help` for the installed version's flags) or `POST /api/v1/repos/<owner>/<repo>/pulls`.

On GitHub, open a stack with `gh stack` instead of chaining `--base` by hand, so GitHub links the PRs as a native stack and retargets them as lower ones merge. Check `gh extension list` for GitHub's `github/gh-stack` extension; if missing, ask the user to install it (`gh extension install github/gh-stack`) rather than installing it yourself, since it runs code from outside the repository with the user's token. Open the PRs as above, bottom first, then link them bottom to top with `gh stack link <pr1> <pr2> ...` using PR numbers. Do not use `gh stack submit`: it has no title or body flags, and in a non-interactive shell it opens draft PRs with auto-generated titles. Without the extension, chain `--base` by hand and say so in the report.

Open stacked PRs bottom first and edit each body once all links exist. Apply labels and reviewers only from repository conventions or the user's instructions. After opening, re-read each PR from the provider to confirm the base, head commit, description, and that the images render.

## Report

List each PR opened with its link, base, size in changed lines, and one-line summary, in merge order. Mention checks run locally and their results, screenshots attached or why none were needed, and anything skipped or left for the user, such as a split the user declined.
