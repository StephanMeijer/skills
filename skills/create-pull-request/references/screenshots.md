# Screenshots for Visual Changes

Screenshots let a reviewer see the result without checking out the branch. Show the change, not the whole app.

## What to Capture

- **Before and after** of every screen, component, or state the PR changes. Capture "before" from the base branch, using a separate worktree (`git worktree add <dir> <base>`) so the PR branch stays untouched.
- **Each affected state**: empty, loading, error, hover or focus, long content, and the states the change was about.
- **Each affected viewport** when layout changed: at least a desktop width and a phone width (around 390 px wide).
- **Light and dark themes** when the change touches colors or theming.
- **CLI or terminal output**: a fenced code block of the real output is usually better than an image, because it is searchable. Use an image only when color or layout is the point.
- **Animations or interactions**: a short GIF or video when a still image cannot show the change.

Use the same viewport, data, zoom, and theme for before and after so the only difference is the change. Crop to the relevant area, keep enough surrounding context to orient the reader, and never capture secrets, tokens, personal data, or production customer data; use seed or fixture data.

## How to Capture

Use whatever the project already has, in this order: a project skill or script for running the app, its end-to-end tooling (Playwright, Cypress, Storybook), or browser automation. With Playwright, for example:

```bash
npx playwright screenshot --viewport-size=1280,800 http://localhost:3000/settings after-settings-desktop.png
npx playwright screenshot --viewport-size=390,844 http://localhost:3000/settings after-settings-mobile.png
```

Name files `<before|after>-<screen>-<viewport>[-<theme>].png`. Keep them in a temporary directory, outside the repository.

Look at every screenshot before uploading it. A blank page, a spinner, a login wall, or an error overlay is not evidence; fix the capture.

## How to Upload

Never commit screenshots to the PR branch; they would end up in the default branch on merge.

### GitHub

GitHub has no API for uploading images into a description. Use the first option that works:

1. **Browser upload**, when browser automation is available and signed in to GitHub: open the PR, edit the description, and attach the files through the editor's file input. GitHub inserts `https://github.com/user-attachments/assets/...` links. Save, then check that the images render.
2. **Assets branch**: commit the images to a dedicated orphan branch, such as `pr-assets`, under a folder per PR (`pr-<number>/`), and reference them by commit SHA so the links never move:

   ```markdown
   ![After, settings, desktop](https://github.com/<owner>/<repo>/blob/<sha>/pr-<number>/after-settings-desktop.png?raw=true)
   ```

   These links work in private repositories for anyone with read access. Use an existing assets branch when the repository has one. Creating a new long-lived branch is visible to the whole team, so ask the user once before creating it.

### Forgejo and Codeberg

Upload each image as an attachment to the PR, then reference the returned `browser_download_url` in the description:

```bash
curl -fsS -X POST -H "Authorization: token $FORGEJO_TOKEN" \
  -F "attachment=@after-settings-desktop.png" \
  "https://<host>/api/v1/repos/<owner>/<repo>/issues/<number>/assets"
```

Do not print or log the token.

## Presenting Them

Put before and after side by side in a table, one row per screen or state:

```markdown
| Before | After |
| --- | --- |
| ![Before, settings, desktop](<url>) | ![After, settings, desktop](<url>) |
```

Give every image alt text that says what it shows. Add a one-line caption under the table when the difference is subtle, such as "The save button no longer overlaps the footer on phones."
