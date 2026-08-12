---
name: programming-python
description: Write, change, review, and test modern Python using uv, pyproject.toml, precise static typing, validated data models, explicit errors, restrained dependencies, and strict local quality gates. Use for Python applications, libraries, packages, CLI tools, services, scripts, and asynchronous code.
---

# Programming Python

Produce modern Python whose types and tests make behavior explicit. Follow the repository's architecture and configured tools before applying these defaults.

## Establish the Contract

1. Inspect `pyproject.toml`, `uv.lock`, supported Python versions, package layout, CI, tests, and repository guidance.
2. Use the newest stable Python supported by project dependencies for applications. For libraries, deliberately choose and test the minimum supported version instead of blindly requiring the newest release.
3. Use `uv` for environments, dependencies, locking, command execution, builds, and publishing. Keep modern configuration in `pyproject.toml`; preserve another established tool only when migration is outside scope.

## Implement Strictly

- Type all production interfaces and meaningful local values precisely. Avoid `typing.Any`, untyped containers, unchecked casts, ignored diagnostics, and annotation-only claims unsupported at runtime.
- Model domain concepts explicitly. Use Pydantic v2 to parse and validate untrusted boundary data; use dataclasses or ordinary typed classes for trusted internal state.
- Parse at system boundaries and keep validated types internally. Do not repeatedly validate loosely shaped dictionaries throughout the codebase.
- Raise precise domain exceptions, preserve exception chains, and catch only errors that can be handled meaningfully. Forbid bare `except`, silent failures, and sentinel returns for genuine errors.
- Prefer the standard library and minimize dependencies. Require a concrete benefit for large, overlapping, weakly maintained, or security-sensitive packages; keep direct dependencies declared and environments locked.
- Keep synchronous code by default. Use async only for genuine concurrent I/O, follow the project's existing async stack, and keep blocking work off the event loop.
- Prefer clear functions and composition over speculative classes, frameworks, decorators, metaprogramming, or compatibility layers. Preserve public API compatibility unless a breaking change is requested.

## Prove the Change Locally

Use the project's testing library and conventions. Add focused tests for changed behavior, boundary validation, and failure paths; prefer real local integrations over mocks when practical.

Before completion, run every applicable gate through `uv` against the final worktree:

1. the repository's own formatting, linting, type-checking, and test commands;
2. Ruff formatting and linting with no ignored new violations;
3. strict `basedpyright` or the project's stricter configured type checker;
4. the complete relevant test suite and configured coverage threshold;
5. package builds and metadata checks for distributable projects;
6. dependency security and policy checks when configured;
7. a real invocation of the changed application, command, library interface, or integration boundary.

Test every supported Python version for libraries when locally practical. Report the exact commands run, behavior exercised, and any gate that could not run with the reason.
