---
name: create-pull-request
description: Open new pull requests on GitHub and Forgejo, including Codeberg, that are small, verified, and easy to review. Splits large changes into several focused PRs (stacked when they depend on each other), writes a description with an ELI5 summary for every PR, and captures and uploads before and after screenshots whenever the change is visual. Use when asked to open, create, raise, or submit a PR or merge request, to turn local commits or a branch into PRs, or to split a large change into smaller PRs. Do not use to review, fix, or rebase an existing PR; use pull-request for that.
license: MIT
---

# Create Pull Request

Turn local work into pull requests a reviewer can understand in minutes and approve with confidence. Three rules apply to every PR this skill opens:

1. **Small.** Prefer several focused PRs over one large one. Each PR does one thing and passes its checks on its own.
2. **ELI5.** Every description opens with a plain-language explanation anyone on the team could follow.
3. **Screenshots when visual.** Any change a user can see ships with screenshots in the description.

## Set the Authority Boundary

Asking to create a PR authorizes pushing the PR branches and opening the PRs. It does not authorize merging, approving, closing other PRs, force-pushing a branch someone else is working on, or requesting reviewers who are not named by the repository's conventions or the user.

- Never push to the default branch or rewrite its history.
- Rewriting local, unpushed commits to split them is allowed. Rewriting already-pushed commits needs the user's approval.
- When the user explicitly asked for a single PR and the change is large, open one PR and say in the report how it could have been split.

## Establish the Ground

1. Read `git remote get-url origin` to tell GitHub (`gh`) from Forgejo or Codeberg (`fj` and the REST API).
2. Find the target branch from the provider's default branch or the user's instruction. Never assume `main`.
3. Inspect `git status`, the current branch, its upstream, and the commits and uncommitted changes relative to the target. Do not sweep unrelated local changes into a PR; ask about them.
4. Read the repository's guidance: `CONTRIBUTING`, `CLAUDE.md`, `AGENTS.md`, the PR template (`.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`, `.forgejo/`, `.gitea/`), commit message conventions, branch naming, and labels.
5. Search open PRs and issues for overlapping work, and find the issues this work resolves.

## Plan Small PRs

Read [references/splitting.md](references/splitting.md) and decide the split before writing any description.

Split when the change contains more than one independent concern: a refactor and a behavior change, a mechanical rename and new logic, a dependency bump and the code that uses it, or features that could ship separately. As a guide, a PR above roughly 400 changed lines, excluding generated files, lockfiles, and snapshots, should be split unless it is one indivisible concern.

State the plan in one short list (PR title, what it contains, what it is based on) and proceed. Each planned PR must build, pass tests, and leave the product working when merged alone, in order.

## Verify Before Opening

For each PR branch, run the repository's build, lint, typecheck, and tests locally, using its documented commands. Do not open a PR whose locally reproducible checks fail; fix the failure or report it. Open as a draft only when the user asks, the repository's conventions say so, or a known failure remains that the user accepted.

## Capture Screenshots for Visual Changes

A change is visual when anything a user sees could look different: UI components, styles, layout, copy, icons, images, generated documents, charts, emails, or CLI output formatting. When in doubt, treat it as visual.

Read [references/screenshots.md](references/screenshots.md) to capture before and after screenshots of every affected screen or state and upload them to the PR description. Never commit screenshots to the PR branch.

## Write the Description

Use [references/description-template.md](references/description-template.md). When the repository has its own PR template, fill that template in and add the ELI5 and Screenshots sections to it rather than replacing it.

- The **ELI5** comes first: two to four short sentences in everyday words, no jargon, no file or function names, saying what changes and why it matters to the person using or running the product. A reviewer who reads nothing else should know what the PR is for.
- The title follows the repository's convention (for example Conventional Commits); otherwise it is a short imperative sentence.
- Link issues with closing keywords (`Closes #123`) only when the PR fully resolves them; otherwise reference them.
- For stacked PRs, list the whole stack with links in every PR and mark which one this is.
- Write the description to a file and pass it with `--body-file`, so quoting and newlines survive.

## Open the PRs

Push each branch with an upstream, then open the PR against its base.

```bash
git push -u origin <branch>
gh pr create --base <base> --head <branch> --title "<title>" --body-file <file>
```

For Forgejo, use `fj pr create` (check `fj pr create --help` for the flags of the installed version) or `POST /api/v1/repos/<owner>/<repo>/pulls`.

Open stacked PRs in order, then edit each body once all links exist. Apply labels and reviewers only from repository conventions or the user's instructions. After opening, re-read each PR from the provider to confirm the base, head commit, description, and that the images render.

## Report

List each PR opened with its link, base, size in changed lines, and one-line summary, in merge order. Mention checks run locally and their results, screenshots attached or why none were needed, and anything skipped or left for the user, such as a split the user declined.
