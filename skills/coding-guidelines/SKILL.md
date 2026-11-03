---
name: coding-guidelines
description: Keep code simple and the architecture sound. Prefer the simplest change that works, remove code that has no effect on functionality or stability, parse input at the boundary, fail loudly with errors as values, prefer no test over a flaky test, and enforce style, formatting, and linting in CI. Use when writing, changing, reviewing, or refactoring code in any language, or when asked about coding guidelines, KISS, simplicity, architecture, input parsing, error handling, flaky tests, or linting.
license: MIT
---

# Coding Guidelines

Simple is good.

## Keep It Simple

- Write the simplest code that does the job. Not the cleverest, not the most general.
- No speculative code: no abstraction, option, or layer until something needs it.
- Boring over clever: plain loops, explicit names, nothing a reader has to stop and decode.
- Fix the root cause once, in the shared code, not with a workaround at every call site.
- Comments say why, not what. If a comment explains what the code does, rewrite the code.

## Architecture Matters

- Simple code, not simple architecture. Keep the parts small, and keep the structure sound.
- Respect the layers, boundaries, and patterns the codebase already has, even where you would have done it differently. Put code where it belongs, not where it is quickest to add. To change a pattern, change it everywhere in a separate change, not one file at a time.
- Dependencies point one way. A lower layer never reaches into a higher one.
- Never flatten or bypass the architecture to save lines.
- Pure functions where possible. Keep logic free of side effects such as I/O, time, and randomness, and push those to the edges.

## Remove Rather Than Add

- Removing code beats adding code. Before adding something, check whether removing something solves it.
- Code that has no effect on functionality or stability goes: dead code, unused options, needless wrappers, defensive checks for cases that cannot happen.
- Use what exists first: the codebase's own helpers, then the standard library, then a dependency already installed. Only then write new code. Never add a dependency for a few lines.
- Never cut validation at trust boundaries, error handling that prevents data loss, or security. For authentication, authorization, tokens, sessions, cookies, or proxy trust, follow the `security-invariants` skill when it is installed.

## Parse, Don't Validate

- Parse untrusted input into a typed value once, at the boundary: requests, files, environment, external APIs. Code past the boundary trusts the type and does not check again.
- Parse with the validator the repository already uses. For a small shape with no existing validator, write a small explicit parser; add the ecosystem's schema library (Zod in TypeScript, Pydantic in Python, serde in Rust, Ecto changesets in Elixir) only when nested schemas, unions, or error reporting make hand-written parsing less trustworthy.
- A failed parse is an error value with the reason, not a half-valid object.

## Errors

- Fail loudly. Never swallow an error or silently fall back to a default.
- Prefer errors as values over exceptions, in the language's idiom: `Result` (`Ok(value)` / `Err(error)`) in Rust, `{:ok, value}` / `{:error, reason}` in Elixir, `(value, error)` in Go. The signature then shows the caller that the call can fail and must be handled. Where the language has no such idiom, use the codebase's existing pattern rather than inventing one.

## Tests

- Test behaviour, not implementation.
- No test is better than a flaky test. A flaky test signals too much complexity, in the test or in the code it tests. Simplify that first; do not paper over it with retries, sleeps, or longer timeouts. If it cannot be made reliable, delete it.

## Style and Tooling

- Style, formatting, and linting are checked automatically in CI/CD, not by reviewers.
- The same formatter and linter run locally and in CI, from one configuration in the repository.
- Fix the code, not the check. Do not disable a lint rule to get a change through.
- Write the guidelines down where every contributor finds them, such as `CONTRIBUTING.md`.
