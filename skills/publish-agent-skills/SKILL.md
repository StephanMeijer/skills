---
name: publish-agent-skills
description: Prepare, publish, and verify public Agent Skills repositories that work with the skills CLI and skills.sh. Use when an agent needs to turn local skill folders into an installable GitHub repository, repair repository discovery, publish new skills, or confirm that a remote skill can be listed and installed with npx skills.
---

# Publish Agent Skills

Publish a small, portable skill repository and prove that consumers can discover it from the public remote.

## Inspect Before Editing

1. Read the working tree, current branch, remotes, and recent history.
2. Identify existing `SKILL.md` files and preserve unrelated user changes.
3. Confirm the target owner, repository name, and public visibility from local context or the authenticated GitHub account.
4. Treat repository creation, pushes, and visibility changes as external writes. Perform them only when the user explicitly requests publication.

## Package Real Skills

- Put each public skill at `skills/<skill-name>/SKILL.md`.
- Match the folder name to the frontmatter `name`.
- Use lowercase letters, digits, and hyphens for names.
- Include only `name` and a trigger-rich `description` in frontmatter unless a consumer requires another standard field.
- Keep operational instructions concise and imperative.
- Bundle only resources the skill actually uses under `scripts/`, `references/`, or `assets/`.
- Never publish TODO text, example placeholders, credentials, local paths, or generated test artifacts.
- Do not invent a placeholder skill merely to make an empty repository appear installable. Ask for the first skill's purpose if no reusable capability can be inferred from the request.

## Make the Repository Legible

Create or update the root `README.md` with:

- a one-sentence repository purpose;
- the available skill names and descriptions;
- `npx skills add <owner>/<repo> --list` for discovery;
- `npx skills add <owner>/<repo> --skill <name>` for installation;
- a short repository layout for contributors.

Do not add npm package metadata merely for the `skills` CLI. The CLI installs skills directly from Git repositories.

Do not choose a software license on the user's behalf. Add one only when the user has selected it or the repository already establishes the choice.

## Validate Locally

Validate every skill folder with the official Agent Skills reference validator when it is available:

```bash
skills-ref validate skills/<skill-name>
```

Then exercise repository discovery without installing anything:

```bash
npx --yes skills@latest add . --list
```

Require the command to exit successfully and show every intended public skill exactly once. Fix the repository structure or frontmatter when the list is empty, duplicated, or incomplete.

## Publish Safely

When the repository does not exist and publication is authorized, create and push it without force:

```bash
gh repo create <owner>/<repo> --public --source=. --remote=origin --push
```

When it already exists:

1. Verify the authenticated account can write to it.
2. Verify that `origin` targets the intended repository.
3. Confirm public visibility.
4. Push normally. Never force-push unless the user explicitly requests history replacement.

Set a concise repository description and relevant topics when authorized to create the repository; these help humans find it but do not replace valid `SKILL.md` files.

## Verify the Public Surface

After the push, list skills from the remote rather than trusting local validation:

```bash
npx --yes skills@latest add <owner>/<repo> --list
```

For an end-to-end check, install one skill into a fresh temporary directory, inspect the installed `SKILL.md`, and remove the temporary directory afterward. Do not install test copies into the user's active project or global agent configuration.

Report the public repository URL, exact install command, discovered skills, checks performed, and any remaining publication caveat. Hosted indexes can update asynchronously; distinguish a verified remote installation from eventual appearance in search results.
