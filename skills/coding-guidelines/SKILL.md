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
- If a reader has to stop and decode it, it is not simple yet.
- Boring over clever: plain loops, explicit names, no one-liners that need decoding.
- Fix the root cause once, in the shared code, not with a workaround at every call site.
- Comments say why, not what. If a comment explains what the code does, rewrite the code.

## Architecture Matters

- Simple code, not simple architecture. Keep the parts small, and keep the structure sound.
- Respect the layers, boundaries, and conventions the codebase already has. Put code where it belongs, not where it is quickest to add.
- Consistency over preference. Follow the patterns the codebase already uses, even where you would have done it differently. To change a pattern, change it everywhere in a separate change, not one file at a time.
- Dependencies point one way. A lower layer never reaches into a higher one.
- Never flatten or bypass the architecture to save lines.
- New structure is added when a real need appears, not in advance.
- Pure functions where possible. Keep logic free of side effects such as I/O, time, and randomness, and push those to the edges. Pure code is easier to reason about and to test, and its tests do not flake.

## Remove Rather Than Add

- Removing code beats adding code.
- Code that has no effect on functionality or stability goes: dead code, unused options, needless wrappers, defensive checks for cases that cannot happen.
- Before adding something, check whether removing something solves it.
- Use what exists first: the codebase's own helpers, then the standard library, then a dependency already installed. Only then write new code. Never add a dependency for a few lines.
- Never cut validation at trust boundaries, error handling that prevents data loss, or security. Simple does not mean unsafe. For authentication, authorization, tokens, sessions, cookies, or proxy trust, follow the `security-invariants` skill when it is installed.

## Parse, Don't Validate

- Parse untrusted input into a typed value once, at the boundary: requests, files, environment, external APIs. Code past the boundary trusts the type and does not check again.
- Parse with the validator the repository already uses. For a small shape with no existing validator, a small explicit parser is fine; add the ecosystem's schema library (Zod in TypeScript, Pydantic in Python, serde in Rust, Ecto changesets in Elixir) only when nested schemas, unions, or error reporting make hand-written parsing less trustworthy.
- A failed parse is an error value with the reason, not a half-valid object.

## Errors

- Fail loudly. Never swallow an error or silently fall back to a default. A clear failure is easier to debug than wrong output.
- Prefer errors as values over exceptions, as in Rust's `Result` (`Ok(value)` / `Err(error)`) or Elixir's `{:ok, value}` / `{:error, reason}`. The caller sees in the signature that a call can fail and has to handle it.
- Follow the language's idiom for this: `Result` in Rust, tagged tuples in Elixir, `(value, error)` in Go. Where the language has no such idiom, use the codebase's existing pattern rather than inventing one.

## Tests

- Test behaviour, not implementation. A test that breaks on every refactor costs more than it protects.
- No test is better than a flaky test.
- A flaky test is a signal of too much complexity, in the test or in the code it tests. Simplify that first; do not paper over it with retries, sleeps, or longer timeouts.
- If it cannot be made reliable, delete it.

## Style and Tooling

- Style, formatting, and linting are checked automatically in CI/CD, not by reviewers.
- The same formatter and linter run locally and in CI, from one configuration in the repository.
- Fix the code, not the check. Do not disable a lint rule to get a change through.
- Write the guidelines down where every contributor finds them, such as `CONTRIBUTING.md`. Clear guidelines help every contributor.
