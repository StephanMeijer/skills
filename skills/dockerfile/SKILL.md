---
name: dockerfile
description: Create, audit, refactor, and optimize production Dockerfiles and .dockerignore files for any common language or framework. Use when asked to containerize an application, write or improve a Dockerfile, reduce image size or attack surface, add multi-stage or multi-platform builds, harden container runtime behavior, fix Docker build problems, or review an existing image definition. Do not use for Docker Compose-only work.
license: MIT
---

# Dockerfile

Create the smallest reproducible production image that preserves the application's real behavior. Work from repository evidence, not language-specific boilerplate.

## Establish the Contract

1. Inspect the complete repository structure, existing Dockerfile variants, `.dockerignore`, lockfiles, build scripts, CI configuration, runtime documentation, and deployment manifests.
2. Determine the actual build command, test command, produced artifacts, startup command, ports, signals, runtime dependencies, configuration inputs, and writable paths.
3. For an existing Dockerfile, build and exercise the current image when practical and record its behavior, user, platforms, size, layers, and scanner findings before changing it, and snapshot its startup configuration as the audit baseline.
4. Preserve supported build arguments, labels, entrypoint semantics, exposed ports, stop signal, and runtime behavior unless the user explicitly requests a breaking change.
5. Keep Docker Compose and deployment configuration out of scope unless separately requested.

If repository evidence does not determine a correctness-critical command or artifact, ask one narrow question; do not guess an ecosystem convention.

Record the contract and every validation gate in a compact evidence ledger: for each item its source or exact command, baseline result when applicable, candidate result, affected stage or platform, and one status: `PASS`, `FAIL`, `UNAVAILABLE`, or `NOT_APPLICABLE`. A missing tool or unsupported environment is never a pass.

## Apply the Image Policy

Read [references/policy.md](references/policy.md) before writing or reviewing a Dockerfile.

Use these priorities in order:

1. correctness and reproducibility;
2. least privilege and minimal attack surface;
3. `linux/amd64` and `linux/arm64` support;
4. small runtime size;
5. effective build caching and readable maintenance.

Choose the runtime base by the policy's base-image routing and pin it by its digest rules; record the reason for the chosen base in the report.

## Separate Build from Runtime

Use named multi-stage builds unless an additional stage provides no isolation, size, cache, or security benefit, with only the stages the repository needs, normally selected from:

```text
dependencies -> build -> test -> runtime
```

Add lint and test stages only when container-specific tests or the repository's existing build design justify them; keep those targets independently buildable and outside the runtime ancestry, and do not duplicate a complete host-side test workflow to add stages. Copy only the final executable or runtime artifact, required shared libraries, explicitly needed certificates/data, and narrowly scoped configuration into the runtime image.

Apply the policy's build-input and caching rules.

## Minimize the Runtime

Apply the policy's runtime hardening rules to the selected base. Prove that the final image contains only required artifacts and system data, starts directly with exec-form arguments, runs as a non-root identity, and works with a read-only root filesystem plus only its demonstrated writable mounts. Every root, tooling, writable-path, or base-image exception is a finding that needs concrete runtime evidence and a documented reason.

## Make Builds Reproducible

Apply the policy's reproducibility and supply-chain rules. Resolve lock modes, base digests, external-artifact verification, OCI metadata, and the multi-platform strategy from repository and publisher evidence. Keep publication-only SBOM, provenance, and push operations out of ordinary local verification.

## Maintain `.dockerignore`

Create or audit `.dockerignore` with every Dockerfile, following the policy's repository-specific procedure: trace the real context and every build input before writing exclusions, never emit a stock pattern list or copy `.gitignore`, and rebuild every target afterwards so a smaller context cannot conceal a missing input.

## Verify Locally

Read [references/verification.md](references/verification.md), then run every applicable local gate before declaring completion. At minimum:

1. run Docker's build checks and Hadolint when installed;
2. build every relevant stage;
3. build for both `linux/amd64` and `linux/arm64` when Buildx can do so;
4. load the native-platform runtime image, start it, and exercise its real interface;
5. verify non-root configuration, read-only-root compatibility, writable paths, dropped capabilities, no-new-privileges compatibility, and graceful shutdown;
6. inspect final size, history, files, packages, labels, and architecture metadata;
7. exercise at least one repository-derived failure path and verify its observable error behavior;
8. scan the final image and build context for vulnerabilities and secrets when applicable tooling is available;
9. repeat a clean build when practical and explain any digest or semantic configuration difference before claiming reproducibility.

For an audit, diff the baseline and candidate startup configuration as described in the verification reference, and compare the remaining before and after evidence. Never accept a behavioral regression for a smaller image. Treat any failed mandatory gate, behavioral regression, unverified required platform, or unverified required runtime path as `INCOMPLETE`, and state exactly why a gate could not run.

Report the files changed, runtime base and justification, final image size, platforms built, checks performed, runtime and failure scenarios observed, scanner results, every unavailable gate, and any remaining exceptions. Report configuration by name and behavior without printing secret values or raw environment dumps.
