---
name: programming-python
description: Write, change, review, and test modern Python using repository-native environments, precise static typing, validated boundaries, explicit errors, restrained dependencies, and complete local quality gates. Use for Python applications, libraries, packages, CLI tools, services, scripts, and asynchronous code.
license: MIT
---

# Programming Python

Produce modern Python whose types and tests make behavior explicit. The repository's architecture and configured tools come before these defaults.

## Establish the Contract

1. Inspect project metadata, every dependency or constraint file, lockfiles, supported Python versions, package layout, entry points, CI, tests, and repository guidance.
2. Classify the project (application, library, service, CLI, package, or script collection) and record its public imports, serialized forms, CLI behavior, exceptions, and other compatibility boundaries before changing them.
3. Use the repository's declared environment, package manager, build backend, and locked commands. Do not infer the interpreter from the machine default, regenerate a lock without an in-scope dependency change, install tools globally, or migrate an established workflow because another tool is preferred.
4. For a new project without established tooling, choose a current maintained workflow from deployment, packaging, and contributor needs; a library declares and tests a minimum Python version, an application follows its declared deployment interpreter.

## Implement Strictly

- Type every changed production interface and meaningful local value precisely: no unrestricted `typing.Any`, bare containers, unchecked casts, ignored diagnostics, or annotations the runtime does not honor.
- Receive untrusted values as `object` and parse JSON, configuration, environment variables, database rows, CLI input, and network data once at the system boundary, with the repository's existing rigorous validator; otherwise a small explicit parser for a small shape, or one justified maintained dependency when nested schemas, unions, refinements, or error reporting make handwritten parsing less trustworthy.
- Convert parsed input into explicit domain values (existing models, dataclasses, enums, tuples, or ordinary typed classes); loose mappings stay at serialization boundaries, out of domain logic. Do not add runtime validation to values already guaranteed by trusted construction.
- Name every meaningful literal: numbers, strings, durations, sizes, limits, keys, and paths live in a module-level `typing.Final` constant or an `enum.Enum`, defined once in the module that owns the concept, and every use refers to that name, never a bare inline literal. Self-evident `0`, `1`, or `""` in a plain arithmetic or identity role are not magic.
- Raise precise domain exceptions, preserve exception chains, and catch only errors that can be handled meaningfully; no bare `except`, silent failures, or sentinel returns for genuine errors.
- Prefer the standard library; add a large, overlapping, weakly maintained, or security-sensitive package only for a concrete benefit. Keep direct dependencies declared and environments locked.
- Stay synchronous; use async only for genuine concurrent I/O, on the project's existing async stack, with blocking work off the event loop.
- Prefer clear functions and composition over speculative classes, frameworks, decorators, metaprogramming, or compatibility layers. Preserve public API compatibility unless a breaking change is requested.
- For a change to authentication, authorization, tokens, sessions, cookies, identity, proxy trust, registration, or abuse controls, follow the `security-invariants` skill when installed: enumerate every sibling entry point before editing, parse scopes, roles, groups, domains, origins, and external identities into explicit domain values, never authorize with substring, prefix, or suffix checks, and add a deterministic regression test that fails for the intended reason before the fix.

## Prove the Change Locally

With the project's testing library and conventions, add focused tests for changed behavior, boundary validation, failure paths, and preserved public behavior; prefer real local integrations over mocks when practical.

Before completion, run every applicable gate through the repository's declared environment against the final worktree:

1. the repository's own formatting, linting, type-checking, and test commands;
2. focused boundary and failure tests, then the complete relevant suite and configured coverage threshold;
3. the repository's strictest configured static type checker, with no new diagnostics ignored;
4. package builds, metadata checks, and clean installation or import checks for distributable projects;
5. dependency, lock, security, and policy checks when configured;
6. a real invocation of the built application, command, public library interface, or integration boundary.

Do not add a formatter, linter, type checker, validator, or packaging migration to satisfy this checklist; a missing gate added separately in scope must be maintained, compatible with the existing workflow, and no weaker than current policy.

For a library, test every supported Python version when locally practical. Report the exact commands run, interpreter versions exercised, behavior observed, baseline failures, and every gate that could not run with its reason; mark conflicting or unavailable evidence as unclear, never as passed.
