---
name: programming-typescript
description: Write, change, review, and test modern TypeScript with strict structural typing, assertion-free domain models, parsed boundaries, modern ESM, restrained dependencies, and complete local quality gates. Use for TypeScript applications, libraries, Node.js, Deno, Bun, frontend projects, Vite, Zod, Effect, and Vitest.
license: MIT
---

# Programming TypeScript

Produce modern TypeScript whose types remain trustworthy at runtime boundaries. Follow the repository's architecture and configured tools before applying these defaults.

## Establish the Contract

1. Inspect `package.json`, lockfiles, `tsconfig` files, runtime and package-manager declarations, build tooling, CI, tests, and repository guidance.
2. Use modern ESM and the newest stable Node.js by default. Choose Deno or Bun only for a concrete project benefit, and follow their recommended package workflow. For libraries, choose and test the supported runtimes.
3. Preserve the established package manager. For new Node.js projects, choose pnpm or npm and commit the matching lockfile.

## Implement Strictly

- Configure TypeScript as strictly as the project can support, and keep enabling checks that expose unsoundness.
- Forbid `any`, type assertions with `as`, angle-bracket assertions, non-null assertions, and ignored type diagnostics. Fix the model, narrow safely, or parse the value instead. Isolate an unavoidable third-party interop escape hatch and prove its invariant.
- Allow `unknown` only at untrusted boundaries, and parse it immediately into a precise domain type. Use the parser the repository already has (Zod, Effect Schema, or equivalent); otherwise write a small explicit parser for a small shape, and add Zod or Effect Schema only when nested schemas, unions, refinements, or error reporting make a handwritten parser less trustworthy.
- Prefer discriminated unions, branded types, readonly data, exhaustive matching, and states that make invalid combinations unrepresentable. Use `satisfies`, annotations, type guards, and validated constructors without lying to the compiler.
- Name every meaningful literal. Numbers, strings, durations, sizes, limits, keys, and paths belong in a top-level `const`, a string-literal union, or a readonly object checked with `satisfies`, defined once in the module that owns the concept, and every use refers to that name. Self-evident values such as `0`, `1`, or `""` in their plain arithmetic or identity role are not magic.
- Use Effect when typed failures, dependency management, concurrency, retries, or resource safety genuinely improve the code; do not wrap simple code ceremonially.
- Prefer platform APIs and minimize dependencies. Require a concrete benefit for large, overlapping, weakly maintained, or security-sensitive packages.
- Keep frontend builds on Vite unless project constraints require another tool. Preserve framework conventions and public API compatibility unless a breaking change is requested.
- For a change to authentication, authorization, tokens, sessions, cookies, identity, proxy trust, registration, or abuse controls, follow the `security-invariants` skill when it is installed: enumerate every sibling entry point before editing, parse scopes, roles, groups, domains, origins, and external identities into branded or validated domain values, never authorize with substring, prefix, or suffix checks (`includes` included), and add a deterministic regression test that fails for the intended reason before the fix.

## Prove the Change Locally

Use Vitest for new tests unless a capable test framework already exists. Add browser or end-to-end coverage only for behavior that requires the real surface.

Before completion, run every applicable gate with the project's package manager against the final worktree:

1. the repository's formatting and linting commands, with no new violations ignored;
2. strict TypeScript checking without emitting output;
3. the complete relevant test suite and configured coverage threshold;
4. production builds for every affected package or application;
5. dependency and package-policy checks when configured;
6. a real invocation of the changed application, library interface, or browser behavior.

Report the exact commands run, the behavior exercised, and any gate that could not run with the reason.
