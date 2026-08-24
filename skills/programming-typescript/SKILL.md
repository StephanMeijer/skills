---
name: programming-typescript
description: Write, change, review, and test modern TypeScript with strict structural typing, assertion-free domain models, parsed boundaries, modern ESM, restrained dependencies, and complete local quality gates. Use for TypeScript applications, libraries, Node.js, Deno, Bun, frontend projects, Vite, Zod, Effect, and Vitest.
license: MIT
---

# Programming TypeScript

Produce modern TypeScript whose types remain trustworthy at runtime boundaries. Follow the repository's architecture and configured tools before applying these defaults.

## Establish the Contract

1. Inspect `package.json`, lockfiles, `tsconfig` files, runtime and package-manager declarations, build tooling, CI, tests, and repository guidance.
2. Use modern ESM and the newest stable Node.js by default. Choose Deno or Bun only for a concrete project benefit; follow their recommended package workflow. For libraries, deliberately choose and test supported runtimes.
3. Preserve the established package manager. For new Node.js projects, prefer pnpm or npm based on project needs; commit the matching lockfile.

## Implement Strictly

- Configure TypeScript as strictly as the project can realistically support. Continually enable checks that expose unsoundness instead of treating one static option list as permanently sufficient.
- Forbid `any`, type assertions with `as`, angle-bracket assertions, non-null assertions, and ignored type diagnostics. Fix the model, narrow safely, or parse the value instead. Isolate an unavoidable third-party interop escape hatch and prove its invariant.
- Allow `unknown` only at genuinely untrusted boundaries. Parse it immediately into a precise domain type with Zod, Effect Schema, or an equally rigorous existing parser.
- Prefer discriminated unions, branded types, readonly data, exhaustive matching, and states that make invalid combinations unrepresentable. Use `satisfies`, annotations, type guards, and validated constructors without lying to the compiler.
- Use Effect when typed failures, dependency management, concurrency, retries, or resource safety provide a genuine improvement; do not wrap simple code ceremonially.
- Prefer platform APIs and minimize dependencies. Require a concrete benefit for large, overlapping, weakly maintained, or security-sensitive packages.
- Keep frontend builds on Vite unless project constraints require another tool. Preserve framework conventions and public API compatibility unless a breaking change is requested.

## Prove the Change Locally

Use Vitest by default for new unit and integration tests, or preserve a capable existing test framework. Add browser or end-to-end coverage only for behavior that requires the real surface.

Before completion, run every applicable local gate with the project's package manager against the final worktree:

1. the repository's formatting and linting commands with no ignored new violations;
2. strict TypeScript checking without emitting output;
3. the complete relevant test suite and configured coverage threshold;
4. production builds for every affected package or application;
5. dependency and package-policy checks when configured;
6. a real invocation of the changed application, library interface, or browser behavior.

Report the exact commands run, behavior exercised, and any gate that could not run with the reason.
