# Description Template

Leave out a section that does not apply, except ELI5, which is always present. Keep the whole description short enough to read in two minutes; the diff holds the details.

```markdown
## ELI5

<Two to four short sentences in everyday words. What was wrong or missing, what this PR changes, and what is different for the person using or running the product. No jargon, no file or function names.>

## What changed

- <One bullet per meaningful change, in the order a reviewer should read the diff.>

## Why

<The problem or goal, and why this approach. Link the issue: `Closes #123` when this PR fully resolves it, `Refs #123` otherwise.>

## Screenshots

| Before | After |
| --- | --- |
| ![<alt text>](<url>) | ![<alt text>](<url>) |

## How to test

1. <Steps a reviewer can follow to see the change work, including setup or seed data.>

<Checks run locally, with their commands and results.>

## Stack

1. #<n> <title>
2. **#<n> <title>** (this PR)

## Notes for reviewers

<Risks, trade-offs, follow-ups left for later PRs, and where to focus attention.>
```

## ELI5 Examples

Good:

> When you changed your email address, the app kept sending receipts to the old one until you logged out and back in. Now the new address is used right away.

> The settings page took about five seconds to open because it loaded every past invoice first. It now loads only the latest ones, so it opens almost instantly.

Too technical; rewrite it:

> Invalidate the `UserProfile` cache in `updateEmail()` so `ReceiptMailer` reads the fresh value.

For internal changes with no visible effect, say what gets easier or safer:

> Two parts of the app each had their own copy of the same date-formatting code, and they had started to disagree. This makes them share one copy, so dates look the same everywhere and future fixes only need to happen once.
