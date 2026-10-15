---
name: programming-go
description: Write, change, review, and test modern Go using the module's declared toolchain, strong defined types, explicit wrapped errors, context-aware and leak-free concurrency, restrained dependencies, and complete local quality gates. Use for Go modules and workspaces, libraries, applications, CLI tools, services, cgo, WASM, and cross-compiled targets.
license: MIT
---

# Programming Go

Produce plain, idiomatic Go whose types, errors, and goroutine lifetimes are explicit and whose changed behavior is proven locally. Follow the repository's established architecture and configured tools before applying these defaults.

## Establish the Contract

1. Inspect `go.mod`, `go.sum`, `go.work`, the `go` and `toolchain` directives, `vendor/`, build tags and constraints, `//go:generate` directives, cgo usage, Makefiles or task runners, CI, linter configuration, tests, examples, and repository guidance.
2. Use the newest stable Go release the module supports. Preserve the declared `go` directive; raising it is a compatibility change for every consumer of a library and needs to be in scope. Applications may track current stable.
3. Identify the module's public packages, exported identifiers, command-line behavior, wire and serialized formats, and error values callers branch on. Treat everything outside `internal/` as public API for a library.
4. Identify applicable `GOOS`/`GOARCH` targets, build tags, and whether cgo is required. Support WASM, cgo, or platform-specific code only when the project requires them; never assume host-only behavior.

Inspect without mutating: do not run `go get`, `go mod tidy`, or `go generate` merely to look around, and do not rewrite `go.sum` or `vendor/` without an in-scope dependency change.

## Implement Strictly

- Model domain concepts with defined types, typed constants, and validated constructors rather than bare `string`, `int`, or `map` values. Represent enumerations as a defined type with `iota` or typed constants, and give every `switch` over one an explicit `default` that returns an error instead of silently falling through.
- Give every meaningful literal a name. Numbers, strings, durations, sizes, limits, keys, paths, and similar values belong in a `const` or typed constant block defined once in the package that owns the concept, and every use refers to that name; a bare literal never appears inline. Self-evident values such as `0`, `1`, or `""` in their plain arithmetic or identity role are not magic.
- Avoid `any` and empty interfaces except at genuinely untrusted or heterogeneous boundaries. Decode JSON, configuration, environment variables, flags, database rows, and network input once at the boundary into precise structs, using `json.Decoder.DisallowUnknownFields` or an equally rigorous existing parser where unknown input is an error, and validate before the value enters domain logic.
- Handle every error. Wrap with context using `fmt.Errorf("...: %w", err)`, expose sentinel errors or error types that callers can match with `errors.Is` and `errors.As`, and never compare error strings. Forbid `_ =` on an error, silent fallback, and logging an error while also returning it.
- Forbid `panic` in production-reachable paths. A panic is acceptable only for a programmer error during initialization that can never depend on input, and the message must state the violated invariant. Do not recover panics to hide bugs.
- Pass `context.Context` as the first parameter of every function that blocks, performs I/O, or starts work that can be cancelled; never store it in a struct or replace a caller's context with `context.Background()`.
- Give every goroutine an owner, a cancellation path, and a guaranteed exit. Close channels only from the sending side, bound concurrency deliberately, and prefer simple sequential code until concurrency is demonstrated to be needed. Protect shared state with the narrowest correct primitive.
- Forbid `unsafe`, `reflect`-driven designs, and cgo when a safe, typed design is practical. When unavoidable, isolate the smallest surface, document every invariant, validate inputs at the safe boundary, and add targeted tests.
- Prefer the standard library, including `net/http`, `log/slog`, `encoding/json`, `errors`, `slices`, and `maps`, and minimize dependencies. Justify large, weakly maintained, duplicated, or security-sensitive modules; keep `go.mod` and `go.sum` tidy and committed.
- Accept interfaces and return concrete types. Define small interfaces in the consuming package, and introduce generics, interfaces, code generation, or abstractions only at real reuse or substitution boundaries. Prefer clear concrete code over speculative flexibility.
- Preserve public API compatibility unless a breaking change is explicitly requested; a breaking change to a v2+ module requires a new major version path. Document exported identifiers with doc comments, including error and concurrency contracts where relevant.

## Prove the Change Locally

Add focused table-driven tests for changed behavior and failure paths using the standard `testing` package or the repository's established test helpers. Use examples, fuzz tests, benchmarks, and golden files only when they test a relevant risk; prefer real local integrations, such as `httptest` servers, over mocks when practical.

Before completion, run every applicable local gate for the final worktree. Reproduce repository-native commands and CI flags first; use generic Go commands only as compatible fallbacks:

1. the repository's own formatting, linting, generation, build, and test commands;
2. `gofmt -l .` (or the configured formatter) reporting no files;
3. `go vet ./...` and `go build ./...` across every affected module and documented build tag;
4. `go test -race ./...` across the same scope, plus the configured coverage threshold;
5. the repository's configured linters, such as `staticcheck` or `golangci-lint`, with no ignored new findings;
6. `go mod tidy -diff` or an equivalent check showing no drift, and `go generate` output matching the committed files when generation is used;
7. vulnerability and policy checks with the repository's configured tooling; propose `govulncheck` only when this gate is missing and adding it is in scope;
8. cross-compilation for each relevant `GOOS`/`GOARCH` and cgo setting;
9. a real invocation of the changed binary, library interface, HTTP endpoint, or integration boundary.

For a non-host target, report compilation separately from execution; run tests only when an appropriate runner, emulator, browser, or device is actually available. Do not silently add a linter, formatter, or tool dependency to satisfy this checklist.

Report the exact commands run, Go version, modules and build tags exercised, targets, behavior observed, baseline failures, and every gate that could not run with the reason. Never upgrade "compiled" to "ran" or one target or tag set to full coverage.
