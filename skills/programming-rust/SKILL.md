---
name: programming-rust
description: Write, change, review, and test Rust code using modern stable Rust, strong types, strict safety, explicit errors, restrained dependencies, and complete local quality gates. Use for Rust projects, Cargo workspaces, crates, libraries, applications, CLI tools, services, embedded or no_std targets, WASM, and FFI.
license: MIT
---

# Programming Rust

Follow the repository's established architecture and supported targets before applying these defaults.

## Establish the Contract

1. Inspect root and member manifests, `Cargo.lock`, `rust-toolchain*`, `.cargo/config*`, workspace inheritance and resolver settings, build scripts, feature definitions, target-specific dependencies and runners, CI, tests, examples, and repository guidance.
2. Use the newest stable Rust and stable language features available to the project; nightly Rust or unstable features need explicit justification.
3. Preserve the declared MSRV. Treat a missing `rust-version` as undeclared, not as current stable. Establish an MSRV for a library without one only when that policy change is in scope and its public dependencies support it; applications may track current stable.
4. Identify applicable targets and feature combinations. Support `no_std`, embedded, WASM, or FFI only when the project requires them; never assume host-only behavior.

Build a constraint ledger for each affected package: toolchain and MSRV, valid feature set, target, runner or execution environment, expected build/test/use gate, source of truth, and availability. Use locked metadata discovery when a lockfile exists, and do not mutate the lock during inspection.

## Implement Strictly

- Model domain concepts with enums, newtypes, and validated constructors; match exhaustively and make invalid states unrepresentable.
- Give every meaningful literal a name: numbers, strings, durations, sizes, limits, keys, and paths belong in a `const`, `static`, or enum defined once in the module that owns the concept, and every use refers to that name. Self-evident values such as `0`, `1`, or `""` in their plain arithmetic or identity role are not magic.
- Express ownership and borrowing directly. Clone only when ownership or measured performance justifies it.
- Return typed, contextual errors through the repository's existing error facilities; add a derive or application-context crate only when it gives a concrete benefit over the standard library and does not duplicate an established dependency. Keep library errors branchable and application-boundary errors actionable.
- Forbid `unwrap`, casual `expect`, `panic!`, unchecked indexing, ignored `Result`s, and silent fallback in production-reachable paths. An `expect` is acceptable only for a proven invariant and must state that invariant.
- Forbid `unsafe` when a safe design is practical. When unavoidable, isolate the smallest unsafe surface, document every safety invariant, validate inputs at the safe boundary, add targeted tests, and run Miri where applicable.
- Prefer `std`; minimize dependencies and enabled features. Justify large, weakly maintained, duplicated, or security-sensitive crates; disable unnecessary default features.
- Keep code synchronous by default. Introduce async only for a demonstrated concurrency or I/O need, on the project's existing runtime rather than prescribing Tokio.
- Introduce traits, generics, macros, and abstractions only at real reuse or substitution boundaries; prefer clear concrete code over speculative flexibility.
- Preserve public API compatibility unless a breaking change is explicitly requested. Document public APIs and their safety, panic, and error contracts.
- For a change to authentication, authorization, tokens, sessions, cookies, identity, proxy trust, registration, or abuse controls, follow the `security-invariants` skill when it is installed: enumerate every sibling entry point before editing; parse scopes, roles, groups, domains, origins, and external identities into newtypes; never authorize with substring, prefix, or suffix checks. If any production profile uses `panic = "abort"`, every input-reachable panic, including one in a background task, terminates the process; add a deterministic regression test that fails for the intended reason before the fix.

## Prove the Change Locally

Add focused unit or integration tests for changed behavior and failure paths. Use doctests, property tests, fuzzing, snapshots, benchmarks, and Miri only when they test a relevant risk.

Before completion, run every applicable local gate on the final worktree, reproducing repository-native commands and CI flags first and using generic Cargo commands only as compatible fallbacks:

1. the repository's own formatting, linting, build, and test commands;
2. `cargo fmt --all --check`;
3. locked `cargo check` and `cargo test` across affected workspace members, targets, and every documented valid feature combination;
4. `cargo clippy` over the same scope with warnings denied;
5. doctests and target-specific builds when relevant;
6. dependency security, license, and policy checks with the repository's configured tooling; propose another maintained tool only when this gate is missing and adding it is in scope;
7. a real invocation of the changed binary, library interface, target artifact, or interoperability boundary.

Use `--all-features` only when that complete combination is declared valid. For a non-host target, report compilation separately from execution, and run tests only when a runner, emulator, browser, device, or hardware path is actually available.

Do not claim an unsupported target, unavailable runtime, or mutually exclusive feature combination was verified. Report the exact command, package, toolchain, feature set, target, exit result, behavior exercised, real-use result, and any gate that could not run with the reason. Never upgrade “compiled” to “ran” or one feature set to matrix coverage.
