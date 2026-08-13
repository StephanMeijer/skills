---
name: dockerfile
description: Create, audit, refactor, and optimize production Dockerfiles and .dockerignore files for any common language or framework. Use when asked to containerize an application, write or improve a Dockerfile, reduce image size or attack surface, add multi-stage or multi-platform builds, harden container runtime behavior, fix Docker build problems, or review an existing image definition. Do not use for Docker Compose-only work.
---

# Dockerfile

Create the smallest reproducible production image that preserves the application's real behavior. Work from repository evidence rather than applying language-specific boilerplate.

## Establish the Contract

1. Inspect the complete repository structure, existing Dockerfile variants, `.dockerignore`, lockfiles, build scripts, CI configuration, runtime documentation, and deployment manifests.
2. Determine the actual build command, test command, produced artifacts, startup command, ports, signals, runtime dependencies, configuration inputs, and writable paths.
3. For an existing Dockerfile, build and exercise the current image when practical. Record its behavior, user, platforms, size, layers, and scanner findings before changing it, and snapshot its startup configuration as the audit baseline.
4. Preserve supported build arguments, labels, entrypoint semantics, exposed ports, stop signal, and runtime behavior unless the user explicitly requests a breaking change.
5. Keep Docker Compose and deployment configuration out of scope unless separately requested.

If repository evidence is insufficient to determine a correctness-critical command or artifact, ask one narrow question. Do not guess a plausible ecosystem convention.

## Apply the Image Policy

Read [references/policy.md](references/policy.md) before writing or reviewing a Dockerfile.

Use these priorities in order:

1. correctness and reproducibility;
2. least privilege and minimal attack surface;
3. `linux/amd64` and `linux/arm64` support;
4. small runtime size;
5. effective build caching and readable maintenance.

Prefer a `scratch` runtime for a completely static artifact that needs no certificates, timezone data, defined user database, shell, or debugging support. Otherwise prefer distroless, then Alpine. Use Debian slim only when compatibility requires it, and document the concrete reason.

Pin production base images with a meaningful publisher tag and the immutable multi-platform index digest. Use trusted official or verified images. Never use `latest`.

## Separate Build from Runtime

Use named multi-stage builds unless an additional stage provides no isolation, size, cache, or security benefit. Use only the stages the repository needs, normally selected from:

```text
dependencies -> build -> test -> runtime
```

When container-specific tests or the repository's existing build design justify lint and test stages, keep those targets independently buildable and outside the runtime ancestry. Do not duplicate a complete host-side test workflow merely to add stages. Copy only the final executable or runtime artifact, required shared libraries, explicitly needed certificates/data, and narrowly scoped configuration into the runtime image.

Use BuildKit cache mounts for dependency caches and secret or SSH mounts for private inputs. Never pass secrets through `ARG`, `ENV`, image labels, copied files, or command-line literals that persist in image metadata.

## Minimize the Runtime

Apply the runtime hardening rules in the policy to the selected base. Prove that the final image contains only required artifacts and system data, starts directly with exec-form arguments, runs as a non-root identity, and works with a read-only root filesystem plus only its demonstrated writable mounts. Treat every root, tooling, writable-path, or base-image exception as a finding that needs concrete runtime evidence and a documented reason.

## Make Builds Reproducible

Apply the reproducibility and supply-chain rules in the policy. Resolve lock modes, base digests, external-artifact verification, OCI metadata, and the actual multi-platform strategy from repository and publisher evidence. Keep publication-only SBOM, provenance, and push operations out of ordinary local verification.

## Maintain `.dockerignore`

Create or audit `.dockerignore` with every Dockerfile by following the repository-specific procedure in the policy. Trace the real context and every build input before writing exclusions; do not emit a stock pattern list or copy `.gitignore`. Rebuild every target after the change so an apparently smaller context cannot conceal a missing input.

## Verify Locally

Read [references/verification.md](references/verification.md), then run every applicable local gate before declaring completion. At minimum:

1. run Docker's build checks and Hadolint when installed;
2. build every relevant stage;
3. build for both `linux/amd64` and `linux/arm64` when Buildx can do so;
4. load the native-platform runtime image, start it, and exercise its real interface;
5. verify non-root configuration, read-only-root compatibility, writable paths, and graceful shutdown;
6. inspect final size, history, files, packages, labels, and architecture metadata;
7. scan the final image with Trivy or Docker Scout when available.

For an audit, diff the baseline and candidate startup configuration as described in the verification reference, and compare the remaining before and after evidence. Never accept a behavioral regression for a smaller image. Treat an unverified required platform or runtime path as incomplete, and state exactly why a gate could not run.

Report the files changed, runtime base and justification, final image size, platforms built, checks performed, runtime scenario observed, scanner results, and any remaining exceptions.
