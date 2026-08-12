# StephanMeijer/skills

Portable [Agent Skills](https://agentskills.io/) for Codex and other skills-compatible agents.

## Available skills

| Skill | Purpose |
| --- | --- |
| `publish-agent-skills` | Prepare, publish, and verify public skill repositories for the `skills` CLI and skills.sh. |

## Install

List the skills in this repository:

```bash
npx skills add StephanMeijer/skills --list
```

Install a skill interactively:

```bash
npx skills add StephanMeijer/skills --skill publish-agent-skills
```

You can target a specific supported agent with `--agent`, or install globally with `--global`.

## Repository layout

Each directory under `skills/` is an independently installable skill:

```text
skills/
└── publish-agent-skills/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

Add new skills as `skills/<skill-name>/SKILL.md`, with matching `name` and `description` frontmatter.
