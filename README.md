# StephanMeijer/skills

Portable [Agent Skills](https://agentskills.io/) for Codex and other skills-compatible agents.

Licensed under the [MIT License](LICENSE).

## Available skills

| Skill | Purpose |
| --- | --- |
| `convert-documents` | Convert office, PDF, markup, ebook, spreadsheet, and presentation formats with CLI tools. |
| `convert-media` | Convert images, audio, and video with CLI tools and verified codec-aware settings. |
| `dockerfile` | Create and audit minimal, secure, reproducible multi-platform Dockerfiles and `.dockerignore` files. |
| `github-issue` | Create and manage GitHub issues with native metadata, relationships, projects, fields, and security workflows. |
| `programming-python` | Write strict, modern Python with precise types, validated models, and complete local checks. |
| `programming-rust` | Write strict, modern Rust with strong types, safe boundaries, and complete local checks. |
| `programming-typescript` | Write strict, modern TypeScript without unsafe assertions or unparsed boundaries. |
| `review-pull-request` | Review GitHub and Forgejo PRs and, when authorized, post findings on the relevant changed code lines. |
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

For document conversion:

```bash
npx skills add StephanMeijer/skills --skill convert-documents
```

For image, audio, and video conversion:

```bash
npx skills add StephanMeijer/skills --skill convert-media
```

For strict, modern Python development:

```bash
npx skills add StephanMeijer/skills --skill programming-python
```

For strict, modern Rust development:

```bash
npx skills add StephanMeijer/skills --skill programming-rust
```

For strict, modern TypeScript development:

```bash
npx skills add StephanMeijer/skills --skill programming-typescript
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
├── convert-documents/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── convert-media/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── dockerfile/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── github-issue/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── programming-python/
│   ├── SKILL.md
│   └── agents/
├── programming-rust/
│   ├── SKILL.md
│   └── agents/
├── programming-typescript/
│   ├── SKILL.md
│   └── agents/
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
