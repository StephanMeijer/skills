# StephanMeijer/skills

Portable [Agent Skills](https://agentskills.io/) for Codex and other skills-compatible agents.

Licensed under the [MIT License](LICENSE).

## Available skills

| Skill | Purpose |
| --- | --- |
| `dockerfile` | Create and audit minimal, secure, reproducible multi-platform Dockerfiles and `.dockerignore` files. |
| `github-issue` | Create and manage GitHub issues with native metadata, relationships, projects, fields, and security workflows. |
| `review-pull-request` | Review GitHub and Forgejo PRs, inspect CI, and reply inside GitHub code-review threads. |
| `ruthless-critic` | Deliver precise, evidence-based, unsparing criticism without personal abuse. |
| `work-on-pull-request` | Fix CI failures, rebase merge conflicts, and address feedback on a PR you own. |

## Install

List the skills in this repository:

```bash
npx skills add StephanMeijer/skills --list
```

For PR review:

```bash
npx skills add StephanMeijer/skills --skill review-pull-request
```

For GitHub issue management:

```bash
npx skills add StephanMeijer/skills --skill github-issue
```

For production Dockerfiles:

```bash
npx skills add StephanMeijer/skills --skill dockerfile
```

For an unsparing critique:

```bash
npx skills add StephanMeijer/skills --skill ruthless-critic
```

For author-side PR follow-up:

```bash
npx skills add StephanMeijer/skills --skill work-on-pull-request
```

You can target a specific supported agent with `--agent`, or install globally with `--global`.

## Repository layout

Each directory under `skills/` is an independently installable skill:

```text
skills/
├── dockerfile/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── github-issue/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── review-pull-request/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── ruthless-critic/
│   ├── SKILL.md
│   └── agents/
└── work-on-pull-request/
    ├── SKILL.md
    ├── agents/
    └── references/
```

Add new skills as `skills/<skill-name>/SKILL.md`, with matching `name` and `description` frontmatter.
