---
name: dockerfile
description: Create, audit, refactor, and optimize production Dockerfiles and .dockerignore files for any common language or framework. Use when asked to containerize an application, write or improve a Dockerfile, reduce image size or attack surface, add multi-stage or multi-platform builds, harden container runtime behavior, fix Docker build problems, or review an existing image definition. Do not use for Docker Compose-only work.
---

# Dockerfile

Create the smallest reproducible production image that preserves the application's real behavior. Work from repository evidence rather than applying language-specific boilerplate.

## Establish the Contract

1. Inspect the complete repository structure, existing Dockerfile variants, `.dockerignore`, lockfiles, build scripts, CI configuration, runtime documentation, and deployment manifests.
2. Determine the actual build command, test command, produced artifacts, startup command, ports, signals, runtime dependencies, configuration inputs, and writable paths.
3. For an existing Dockerfile, build and exercise the current image when practical. Record its behavior, user, platforms, size, layers, and scanner findings before changing it.
4. Preserve supported build arguments, labels, entrypoint semantics, exposed ports, and runtime behavior unless the user explicitly requests a breaking change.
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

- Run as the base image's established non-root identity or a fixed numeric UID/GID such as `10001:10001`. Run as root only when fundamentally required; document why and minimize the duration, capabilities, ownership changes, and writable paths.
- Make the root filesystem read-only compatible whenever practical. Declare only the exact writable directories the application needs, including `/tmp` only when used.
- Do not install source code, tests, caches, package indexes, compilers, headers, VCS tools, shells, editors, process monitors, `curl`, `wget`, or package managers into the final stage unless a runtime requirement explicitly justifies them. Prefer distroless or `scratch` when the final image must contain no shell or package manager. When Alpine is deliberately selected, treat its built-in BusyBox and `apk` as an explicit base-image tradeoff rather than claiming they are absent.
- Add CA certificates, timezone data, locales, and native libraries only when runtime evidence requires them.
- Use exec-form `ENTRYPOINT` and `CMD`. Run the application directly as PID 1; add a minimal init only when the process cannot forward signals or reap children correctly.
- Add `EXPOSE` only for a stable documented port. Omit `HEALTHCHECK` by default because probes usually belong to deployment configuration; preserve a deliberate existing health check.
- Use narrow `COPY --chown` operations instead of broad recursive `chown` or `chmod` commands.

## Make Builds Reproducible

- Require lockfiles and frozen or locked dependency installation modes where the project supports them.
- Copy dependency manifests before frequently changing source files, then use cache mounts without retaining cache data in image layers.
- Avoid blanket operating-system upgrades. Select an updated pinned base instead.
- Verify every externally downloaded artifact with a pinned checksum or trusted signature. Prefer `ADD --checksum` for a fixed HTTPS artifact or perform verification in a disposable builder stage.
- Add OCI source, revision, version, and license labels when their real values are available. Omit creation-time labels because they make otherwise identical builds differ. Never commit placeholder metadata.
- Support both required architectures without hard-coded host architecture assumptions. Use BuildKit's platform arguments only when cross-compilation actually needs them.
- Generate SBOM and provenance attestations in an authorized Buildx publication workflow. Do not push merely to validate a Dockerfile.

## Maintain `.dockerignore`

Create or audit `.dockerignore` with every Dockerfile. Exclude VCS metadata, local dependencies, build outputs, caches, logs, editor files, credentials, environment files, coverage, temporary artifacts, and anything else not consumed by the build.

Derive exclusions from the actual build. Do not blindly exclude documentation, tests, generated code, workspace manifests, or VCS metadata when a build or versioning step reads them. Prefer a small context over a large denylist that accidentally includes sensitive files.

## Verify Locally

Read [references/verification.md](references/verification.md), then run every applicable local gate before declaring completion. At minimum:

1. run Docker's build checks and Hadolint when installed;
2. build every relevant stage;
3. build for both `linux/amd64` and `linux/arm64` when Buildx can do so;
4. load the native-platform runtime image, start it, and exercise its real interface;
5. verify non-root configuration, read-only-root compatibility, writable paths, and graceful shutdown;
6. inspect final size, history, files, packages, labels, and architecture metadata;
7. scan the final image with Trivy or Docker Scout when available.

For an audit, compare before and after evidence. Never accept a behavioral regression for a smaller image. Treat an unverified required platform or runtime path as incomplete, and state exactly why a gate could not run.

Report the files changed, runtime base and justification, final image size, platforms built, checks performed, runtime scenario observed, scanner results, and any remaining exceptions.
