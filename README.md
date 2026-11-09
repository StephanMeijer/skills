# StephanMeijer/skills

Portable [Agent Skills](https://agentskills.io/) for Codex and other skills-compatible agents.

Licensed under the [MIT License](LICENSE).

## Available skills

| Skill | Purpose |
| --- | --- |
| `batch-issues` | Gather open issues from any tracker, find the ones cheaper to do together, and offer one to three batches. |
| `batch-plan` | Plan a task or a batch of issues before coding, with shared groundwork once and at least one PR per item. |
| `champion-challenger` | Search for the best classifier: one radical idea per round, scored against the champion, losers salvaged. |
| `coding-guidelines` | Keep code simple and the architecture sound: remove rather than add, parse at the boundary, errors as values, no flaky tests, linting in CI. |
| `convert-documents` | Convert office, PDF, markup, ebook, spreadsheet, and presentation formats with CLI tools. |
| `convert-media` | Convert images, audio, and video with CLI tools and verified codec-aware settings. |
| `create-pull-request` | Open small, verified PRs, split by concern, each with an ELI5 and screenshots for visual changes. |
| `dockerfile` | Create and audit minimal, secure, reproducible multi-platform Dockerfiles and `.dockerignore` files. |
| `email-campaign` | Build an evidence-based case and get companies, officials, allies, and press to engage by e-mail. |
| `git-commit` | Commit as one GPG-signed, signed-off Conventional Commit with a short subject line and no body. |
| `github-actions` | Write, harden, and debug GitHub Actions workflows: SHA-pinned via `gh actions-lock`, least privilege, injection-safe. |
| `github-issue` | Create and manage GitHub issues with native metadata, relationships, projects, fields, and security workflows. |
| `merge-pull-request` | Merge a PR by rebase, squash, or merge commit with an explicit message, so each change lands as exactly one changelog entry, then clean up its branch and worktrees. |
| `prod-hunt` | Hunt what blocks production readiness, prove each blocker, and report a ready or not-ready verdict with ranked fixes. |
| `programming-go` | Write strict, modern Go with explicit errors, owned concurrency, and complete local checks. |
| `programming-python` | Write strict, modern Python with precise types, validated models, and complete local checks. |
| `programming-rust` | Write strict, modern Rust with strong types, safe boundaries, and complete local checks. |
| `programming-typescript` | Write strict, modern TypeScript without unsafe assertions or unparsed boundaries. |
| `pr-steward` | Loop over every open GitHub PR with sub-agents that review, fix, and rebase, report or automerge ready PRs, clean up worktrees and branches of merged PRs, and stop the loop when nothing is left. |
| `pull-request` | Review GitHub and Forgejo PRs, post findings on changed code lines, and drive a PR toward merge as its author. |
| `refactor-debts-and-smells` | Refactor technical debt and code smells, magic values, and scattered literals, or file issues for larger rewrites. Better, not compact. |
| `ruthless-critic` | Deliver precise, evidence-based, unsparing criticism without personal abuse. |
| `security-invariants` | Design and audit trust-boundary invariants, path matrices, adversarial sequences, and regression-tested fixes. |
| `shopping` | Compare products, offers, and reviews across the wider web before buying. |
| `squash-pull-request` | Rebase a PR onto its base and squash it into one GPG-signed, signed-off commit with a subject line only. |
| `support-ticket` | Advance technical support cases with falsifiable tests, evidence analysis, and actionable provider updates. |

## Install

List the skills in this repository:

```bash
npx skills add StephanMeijer/skills --list
```

Install one skill by the name in the table above:

```bash
npx skills add StephanMeijer/skills --skill <name>
```

You can target a specific supported agent with `--agent`, or install globally with `--global`.

## Recommended GitHub CLI extensions

The pull request and GitHub Actions skills use two official GitHub CLI extensions when they are installed, and ask you to install them when they are needed:

```bash
gh extension install github/gh-stack         # native stacked pull requests: link, rebase, and retarget a stack
gh extension install github/gh-actions-lock  # pin every GitHub Action to a commit SHA in .github/workflows/actions.lock
```

## Repository layout

Each directory under `skills/` is an independently installable skill: `skills/<name>/SKILL.md` with `name` and `description` frontmatter (`name` matches the directory), `agents/openai.yaml`, and an optional `references/` directory.
