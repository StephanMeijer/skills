---
name: refactor-technical-debt
description: Act as a senior engineer to find code that can do the same thing with less, better code (duplication, reinvented library features, needless layers, dead or defensive code) and either rewrite it in place under passing tests or file GitHub issues for larger rewrites. Better, not compact; never trades clarity for fewer lines. Use for tech-debt sweeps, simplification passes, "this could be simpler", or turning refactor ideas into issues. Do not use for bug hunting or feature work.
license: MIT
---

# Refactor Technical Debt

Find code that does its job with more machinery than the job needs, and replace it with less, better code that does exactly the same thing. Work as a senior engineer whose rewrites a strong reviewer would approve on sight: fewer concepts, fewer branches, fewer copies, less hand-rolled machinery, clearer flow.

The goal is **better code, not compact code**. Code shrinks here because something was removed: a duplicate, a dead branch, a wrapper that adds nothing, a reimplementation of the standard library. It never shrinks because statements were squeezed together. A rewrite that saves lines by nesting ternaries, chaining one-liners, shortening names, or reaching for clever tricks is a regression, whatever the line count says. The test for every change: would a strong reviewer call the result clearer and easier to change?

## Set the Authority Boundary

Determine whether the user wants a report, in-place fixes, filed issues, or both.

- Reporting is read-only.
- Editing code and filing issues require the user to have asked for them. "Clean this up" or "fix it" authorizes edits; "file issues" or "make tickets" authorizes issues. When unclear, report first and ask.
- Never commit, push, open a pull request, or close an issue unless asked.
- File issues with the conventions of the `github-issue` skill when it is installed; otherwise use `gh issue create` directly.

## Scope the Sweep

Use the path, module, or area the user names; do not widen it. Without one, sweep the whole repository, but start where debt costs most: rank files by churn (`git log --format= --name-only | sort | uniq -c | sort -rn`) and read the hot spots first. Skip generated, vendored, lockfile, migration, and fixture content.

Before judging anything, learn the repository's idioms: read `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING`, linter and formatter configuration, and ADRs. A rewrite that fights an established convention is not an improvement, even when it would be in another codebase. Find the exact build, test, lint, and typecheck commands; they are the safety net for every fix.

## Find Candidates

Let tools find the mechanical debt, then read the code for the rest.

- Run what the ecosystem offers in check mode: duplication detectors (`jscpd`), dead-code finders (`knip`, `ts-prune`, `vulture`, `cargo udeps`, `deadcode`), and the linter's complexity and redundancy rules.
- Read the hot spots for the patterns in [references/smells-and-rewrites.md](references/smells-and-rewrites.md): logic duplicated with small variations, hand-rolled versions of standard or already-installed library features, pass-through layers, defensive code for states the types already exclude, stored state that could be derived, flag parameters, and conditional ladders a table or type would replace.

Record each candidate with evidence: file and line range, the current shape, the proposed shape as a short sketch, and what concretely gets removed.

## Judge Every Candidate

Keep a candidate only when both hold:

1. it **removes** something real: a concept, a branch, a copy, a dependency, a layer, or hand-written code that a standard facility already provides; and
2. the result is **at least as readable** to someone new to the code, and easier to change.

Reject a candidate when it:

- only compacts code (see the anti-patterns in [references/smells-and-rewrites.md](references/smells-and-rewrites.md));
- swaps one acceptable style for another, or is a matter of taste;
- removes a name, type, or abstraction that carries domain meaning;
- changes observable behavior, including errors, ordering, logging, performance characteristics, or public API, unless the user asked for that;
- contradicts a recorded decision in an ADR or the repository's conventions.

Rank survivors by value: how much complexity goes away, multiplied by how often that code is read or changed.

## Triage: Fix Now or File an Issue

**Fix now** when the rewrite is local (one module, no public interface change), the affected behavior is covered by tests that pass today, and the risk is low.

**File an issue** when the rewrite crosses modules, changes a public or exported interface, touches code no test covers, needs a design decision, or is too large to review as one focused change. When coverage is missing, the issue names the characterization test to write before the rewrite.

When the user asked only for fixes, list issue-sized candidates in the report instead of filing them. When they asked only for issues, file every surviving candidate.

## Fix Loop

1. Run the baseline: tests, lint, and typecheck. If the baseline fails, report it and fix nothing that the failing checks cover.
2. Apply one rewrite at a time, matching the surrounding code's style.
3. Rerun the checks. Revert the rewrite if anything fails, or if the diff on rereading is not clearly better than the original.
4. Never edit a test's assertions to make a rewrite pass. A test that must change means the behavior changed; revert and file it as an issue instead.
5. Remove code that the rewrite made dead, such as now-unused helpers, imports, and configuration.

## Filing Issues

Search open issues for an existing report first. File one issue per independent rewrite using [references/issue-template.md](references/issue-template.md); group issues under a parent only when they share one design decision. Write each issue so an engineer with no context from this session can pick it up cold.

## Report

Keep the report proportional and concrete:

- **Fixed**: each rewrite with file references, what it removed, and the checks run afterwards with their results;
- **Filed**: each issue with its link and one-line summary;
- **Proposed**: issue-sized candidates not filed, when issues were not authorized;
- **Rejected**: notable candidates dropped, with a one-line reason each;
- **Blockers**: failing baselines, missing tooling, or areas skipped.

Report the net line change as a fact, not as the measure of success.
