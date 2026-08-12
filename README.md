# StephanMeijer/skills

Portable [Agent Skills](https://agentskills.io/) for Codex and other skills-compatible agents.

Licensed under the [MIT License](LICENSE).

## Available skills

| Skill | Purpose |
| --- | --- |
| `publish-agent-skills` | Prepare, publish, and verify public skill repositories for the `skills` CLI and skills.sh. |
| `review-pull-request` | Review GitHub and Forgejo PRs, inspect CI, and reply inside GitHub code-review threads. |
| `work-on-pull-request` | Fix CI failures, rebase merge conflicts, and address feedback on a PR you own. |

## Install

List the skills in this repository:

```bash
npx skills add StephanMeijer/skills --list
```

Install a skill interactively:

```bash
npx skills add StephanMeijer/skills --skill publish-agent-skills
```

For PR review:

```bash
npx skills add StephanMeijer/skills --skill review-pull-request
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
├── publish-agent-skills/
│   ├── SKILL.md
│   └── agents/
├── review-pull-request/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
└── work-on-pull-request/
    ├── SKILL.md
    ├── agents/
    └── references/
```

Add new skills as `skills/<skill-name>/SKILL.md`, with matching `name` and `description` frontmatter.
