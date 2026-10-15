---
name: refactor-debts-and-smells
description: Act as a senior engineer to find technical debt and code smells (duplication, magic numbers and strings, values scattered across files instead of named constants, reinvented library features, needless layers, dead or defensive code, long functions and parameter lists, primitive obsession, misleading names) in phased sweeps run by parallel sub-agents, and either refactor them in place under passing tests, delivered as many tiny pull requests, or file GitHub issues for larger rewrites. Better, not compact; never trades clarity for fewer lines. Use for tech-debt sweeps, code-smell hunts, simplification passes, "this needs refactoring", "no magic numbers", or turning refactor ideas into issues. Do not use for bug hunting or feature work.
license: MIT
---

# Refactor Debts and Smells

Find technical debt and code smells: code that does its job with more machinery than the job needs, knowledge copied into several places, and values or names that hide what the code means. Replace it with better code that does exactly the same thing. Work as a senior engineer whose refactors a strong reviewer would approve on sight: fewer concepts, fewer branches, fewer copies, one owner for every value, less hand-rolled machinery, clearer flow.

The goal is **better code, not compact code**. Code often shrinks because something was removed: a duplicate, a dead branch, a wrapper that adds nothing, a reimplementation of the standard library. Sometimes it grows slightly, because a magic value gained a name or a smell gained a type. It never shrinks because statements were squeezed together. A rewrite that saves lines by nesting ternaries, chaining one-liners, shortening names, or reaching for clever tricks is a regression, whatever the line count says. The test for every change: would a strong reviewer call the result clearer and easier to change?

## Set the Authority Boundary

Determine whether the user wants a report, in-place fixes, filed issues, or both.

- Reporting is read-only.
- Editing code and filing issues require the user to have asked for them. "Clean this up" or "fix it" authorizes edits; "file issues" or "make tickets" authorizes issues. When unclear, report first and ask.
- Never commit, push, open a pull request, or close an issue unless asked. When the user authorizes delivering the fixes, the preferred outcome is many tiny pull requests, as described in [Deliver as Tiny Pull Requests](#deliver-as-tiny-pull-requests).
- File issues with the conventions of the `github-issue` skill when it is installed; otherwise use `gh issue create` directly.

## Run the Sweep in Phases

Run every sweep as explicit phases, and fan the independent work in each phase out to parallel sub-agents. A single agent reading a whole repository loses detail; several focused agents with fresh context each find more, and keep their file dumps out of the orchestrator's context. When the harness offers a workflow or phase orchestration feature and the user has opted into it, express these phases as a workflow; otherwise launch the parallel sub-agents of each phase together in one message. Only when the scope is a single small module, or the harness has no sub-agents, run the phases sequentially yourself.

1. **Recon (orchestrator).** Scope the sweep, learn the conventions, find the check commands, and run the baseline, as described below. Split the scope into slices that do not overlap: by smell family (duplication and scattered values; reinvented library features and needless layers; dead and defensive code; long functions, parameter lists, and primitive obsession; misleading names), by module or directory, or both for a large repository. Hand every agent the same brief: the scope, the conventions, the check commands, the judgment rules in this skill, and the candidate record format.
2. **Hunt (parallel, read-only).** One agent per slice finds candidates and returns them as records only, never edits. Duplication and repeated literals cross slice boundaries, so give one agent the whole scope for those rather than splitting them by directory.
3. **Judge (orchestrator, optionally parallel).** Merge the records, drop duplicates found by several agents, and apply [Judge Every Candidate](#judge-every-candidate). For a large set, give each surviving candidate to an independent verifier agent instructed to argue against it; keep only candidates that survive.
4. **Fix (parallel, isolated).** Assign each fix-now candidate to an agent that owns a set of files no other agent touches, working in its own git worktree on its own branch cut from the up-to-date default branch when the harness supports isolation. Each agent follows the [Fix Loop](#fix-loop) and returns its branch with the checks it ran. When pull requests are authorized, each branch becomes its own tiny pull request; otherwise the orchestrator integrates the diffs one at a time and reruns the full checks after each. A diff that conflicts or fails is reverted and reported, not patched over. Candidates that touch the same files go to the same agent, in sequence, still as separate commits and, where they are independent, separate pull requests.
5. **File (parallel).** File each issue-sized candidate with its own agent when there are many; the orchestrator first checks for existing issues so two agents never file the same one.
6. **Report (orchestrator).** Combine the results into one report, attributing nothing to a phase that did not actually run.

Sub-agents inherit this skill's authority boundary: a hunt agent never edits, and a fix or file agent acts only when the user authorized fixes or issues.

## Scope the Sweep

Use the path, module, or area the user names; do not widen it. Without one, sweep the whole repository, but start where debt costs most: rank files by churn (`git log --format= --name-only | sort | uniq -c | sort -rn`) and read the hot spots first. Skip generated, vendored, lockfile, migration, and fixture content.

Before judging anything, learn the repository's idioms: read `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING`, linter and formatter configuration, and ADRs. A rewrite that fights an established convention is not an improvement, even when it would be in another codebase. Find the exact build, test, lint, and typecheck commands; they are the safety net for every fix.

## Find Candidates

Let tools find the mechanical debt, then read the code for the rest.

- Run what the ecosystem offers in check mode: duplication detectors (`jscpd`), dead-code finders (`knip`, `ts-prune`, `vulture`, `cargo udeps`, `deadcode`), magic-number rules (`no-magic-numbers`, `PLR2004`, `clippy::unreadable_literal`), and the linter's complexity and redundancy rules.
- Search for repeated literals: the same number, string, URL, path, key, status, or timeout appearing in more than one place is knowledge without an owner. For example, `grep -rnoE '"[^"]{3,}"' <src> | cut -d: -f3- | sort | uniq -c | sort -rn | head`, adapted to the language.
- Read the hot spots for the patterns in [references/smells-and-rewrites.md](references/smells-and-rewrites.md): logic duplicated with small variations, magic and scattered values, hand-rolled versions of standard or already-installed library features, pass-through layers, defensive code for states the types already exclude, stored state that could be derived, flag parameters, conditional ladders a table or type would replace, long functions and parameter lists, primitive obsession, and misleading names.

Record each candidate with evidence: file and line range, the current shape, the proposed shape as a short sketch, and what concretely gets removed.

## Judge Every Candidate

Keep a candidate only when both hold:

1. it **removes** something real: a concept, a branch, a copy (including a copied literal), a dependency, a layer, or hand-written code that a standard facility already provides; or it **names** something real that is currently implicit, such as a magic value or a primitive that stands for a domain concept; and
2. the result is **at least as readable** to someone new to the code, and easier to change.

Reject a candidate when it:

- only compacts code (see the anti-patterns in [references/smells-and-rewrites.md](references/smells-and-rewrites.md));
- swaps one acceptable style for another, or is a matter of taste;
- removes a name, type, or abstraction that carries domain meaning;
- turns a self-explanatory literal into a constant that adds no meaning (see the trap under magic values in [references/smells-and-rewrites.md](references/smells-and-rewrites.md));
- changes observable behavior, including errors, ordering, logging, performance characteristics, or public API, unless the user asked for that;
- contradicts a recorded decision in an ADR or the repository's conventions.

Rank survivors by value: how much complexity goes away, multiplied by how often that code is read or changed.

## Triage: Fix Now or File an Issue

**Fix now** when the rewrite is local (one module, or extracting scattered literals into constants without changing their values; no public interface change), the affected behavior is covered by tests that pass today, and the risk is low.

**File an issue** when the rewrite crosses modules, changes a public or exported interface, touches code no test covers, needs a design decision, or is too large to review as one focused change. When coverage is missing, the issue names the characterization test to write before the rewrite.

When the user asked only for fixes, list issue-sized candidates in the report instead of filing them. When they asked only for issues, file every surviving candidate.

## Fix Loop

1. Run the baseline: tests, lint, and typecheck. If the baseline fails, report it and fix nothing that the failing checks cover.
2. Apply one rewrite at a time, matching the surrounding code's style.
3. Rerun the checks. Revert the rewrite if anything fails, or if the diff on rereading is not clearly better than the original.
4. Never edit a test's assertions to make a rewrite pass. A test that must change means the behavior changed; revert and file it as an issue instead.
5. Remove code that the rewrite made dead, such as now-unused helpers, imports, and configuration.

## Deliver as Tiny Pull Requests

When the user authorizes pull requests, deliver the fixes as many tiny pull requests rather than one large one. A reviewer approves a five-line, one-idea refactor in a minute; a sweep bundled into one pull request stalls, and one doubtful change blocks every good one.

- One pull request per rewrite: one duplicate removed, one constant extracted, one layer inlined, one misleading name fixed. Split further whenever a pull request would need two sentences to explain what it does.
- Cut each branch from the up-to-date default branch so the pull requests are independent and can merge in any order. Stack a pull request on another only when it genuinely depends on it, and say so in the description.
- Each pull request passes the full checks on its own and changes no behavior; its description names the smell, what was removed or named, and the checks run.
- Open them with the conventions of the `create-pull-request` skill when it is installed; otherwise use `gh pr create` directly.
- Never merge, approve, or close a pull request.

## Filing Issues

Search open issues for an existing report first. File one issue per independent rewrite using [references/issue-template.md](references/issue-template.md); group issues under a parent only when they share one design decision. Write each issue so an engineer with no context from this session can pick it up cold.

## Report

Keep the report proportional and concrete:

- **Fixed**: each rewrite with file references, what it removed, and the checks run afterwards with their results;
- **Opened**: each pull request with its link and one-line summary, and any stacking between them;
- **Filed**: each issue with its link and one-line summary;
- **Proposed**: issue-sized candidates not filed, when issues were not authorized;
- **Rejected**: notable candidates dropped, with a one-line reason each;
- **Blockers**: failing baselines, missing tooling, or areas skipped.

Report the net line change as a fact, not as the measure of success.
