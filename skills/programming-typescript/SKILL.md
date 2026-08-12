---
name: programming-typescript
description: Write, change, review, and test modern TypeScript with strict structural typing, truthful domain models, parsed runtime boundaries, restrained dependencies, preserved ecosystem conventions, and complete local quality gates. Use for TypeScript applications, libraries, Node.js, Deno, Bun, frontend and backend projects, packages, browser code, and asynchronous services.
---

# Programming TypeScript

Produce modern TypeScript whose types remain trustworthy at runtime boundaries. Follow the repository's architecture and configured tools before applying these defaults.

## Establish the Contract

1. Inspect package manifests, runtime and package-manager declarations, every lockfile, compiler versions and complete configuration inheritance, module type and export conditions, build tooling, CI, tests, runtime entry points, and repository guidance.
2. Preserve the established runtime, module system, package manager, lockfile, framework, build stack, and public API unless the request explicitly changes them. Treat conflicting declarations or lockfiles as a blocker to resolve, not permission to choose or regenerate one.
3. For a new project without established conventions, choose the runtime, module format, package workflow, and supported-version policy deliberately from deployment and consumer needs. For libraries, build and test every declared runtime and export condition that is affected.

## Implement Strictly

- Configure affected TypeScript as strictly as the project can realistically support. Run a baseline type check before broad configuration changes, never weaken compiler settings, and keep an unrelated monorepo-wide strictness migration outside the current change.
- Do not introduce unrestricted `any`, unchecked value assertions, forced non-null access, casts through `unknown`, or ignored production diagnostics. Fix the model, narrow with runtime facts, or parse the value instead. Checking and modeling constructs such as `satisfies` and `as const` are allowed when they do not fabricate a runtime value. Isolate an unavoidable third-party interop escape hatch and prove its invariant.
- Accept genuinely untrusted input as `unknown` and parse it immediately at ingress. Prefer the repository's existing rigorous parser. Otherwise write a small explicit parser for a small shape or justify one maintained schema dependency when nested objects, unions, refinements, versioning, or structured errors make handwritten validation less trustworthy. Define unknown-key, coercion, default, and nullability behavior explicitly.
- Prefer discriminated unions, branded types, readonly data, exhaustive matching, and states that make invalid combinations unrepresentable. Use `satisfies`, annotations, type guards, and validated constructors without lying to the compiler.
- Add an effect or resource framework only when demonstrated typed-failure composition, dependency provisioning, structured concurrency, retries, cancellation, or lifetime safety outweighs its concepts and migration cost. Prefer an already established framework; otherwise use ordinary functions, explicit results or errors, `try/finally`, and supported platform cancellation primitives.
- Prefer platform APIs and minimize dependencies. Require a concrete benefit for large, overlapping, weakly maintained, or security-sensitive packages.
- Preserve frontend and server framework conventions, build behavior, module exports, and public API compatibility unless a breaking change is requested.

## Prove the Change Locally

Preserve a capable existing test framework. For new work, choose the smallest maintained runner that supports the declared runtime and required behavior. Add browser or end-to-end coverage only for contracts that require a real browser surface.

Before completion, run every applicable local gate with the project's package manager against the final worktree:

1. the repository's formatting and linting commands with no ignored new violations;
2. strict TypeScript checking without emitting output;
3. the complete relevant test suite and configured coverage threshold;
4. production builds for every affected package or application;
5. dependency and package-policy checks when configured;
6. a consumer import through the built public package or a real invocation of the built application;
7. real browser behavior when the changed contract depends on DOM, navigation, hydration, storage, network, accessibility, or browser APIs.

Keep type checking distinct from execution: a runtime that can execute type-stripped source does not prove that the compiler accepts it or that the production build/export works.

Report the exact commands run, runtime and module conditions exercised, behavior observed, baseline failures, and any gate that could not run with the precise reason.
