# Issue Template

Title: a short imperative sentence naming the rewrite and what it removes, such as "Replace hand-rolled retry loop in `sync/` with the HTTP client's retry policy".

Body:

```markdown
## Current shape

<file:line references, and a short description or excerpt of the code as it is now.>

## Proposed shape

<A short sketch of the code after the rewrite. Enough to show the idea, not a full patch.>

## Why it is better

<What is removed (a concept, a branch, a copy, a layer, or reinvented machinery) and why the result is clearer and easier to change. Not "fewer lines".>

## Behavior to preserve

<Observable behavior that must not change: return values, errors, ordering, side effects, logging, public API, and performance characteristics that matter.>

## Verification

<Existing tests that cover it, and any characterization test to write *before* the rewrite when coverage is missing. The exact commands to run.>

## Risk

<Blast radius: callers, modules, and consumers affected, plus anything that could depend on the current behavior.>
```

Labels: use the repository's existing label for refactoring or technical debt when there is one. Do not create new labels without asking.
