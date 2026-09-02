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
| `pull-request` | Review GitHub and Forgejo PRs, post findings on changed code lines, and drive a PR toward merge as its author. |
| `ruthless-critic` | Deliver precise, evidence-based, unsparing criticism without personal abuse. |
| `shopping` | Compare products, offers, and reviews across the wider web before buying. |
| `support-ticket` | Advance technical support cases with falsifiable tests, evidence analysis, and actionable provider updates. |

## Install

List the skills in this repository:

```bash
npx skills add StephanMeijer/skills --list
```

For pull request review and author-side work:

```bash
npx skills add StephanMeijer/skills --skill pull-request
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

For product and shopping research:

```bash
npx skills add StephanMeijer/skills --skill shopping
```

For advancing technical support tickets:

```bash
npx skills add StephanMeijer/skills --skill support-ticket
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
├── pull-request/
│   ├── SKILL.md
│   ├── agents/
│   └── references/
├── ruthless-critic/
│   ├── SKILL.md
│   └── agents/
├── shopping/
│   ├── SKILL.md
│   └── agents/
└── support-ticket/
    ├── SKILL.md
    ├── agents/
    └── references/
```

Add new skills as `skills/<skill-name>/SKILL.md`, with matching `name` and `description` frontmatter.
