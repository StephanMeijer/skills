# Dockerfile Policy

## Contents

- Base-image routing
- Stages and artifacts
- Build inputs and caching
- Runtime hardening
- Reproducibility and supply chain
- Instruction rules
- `.dockerignore`

## Base-Image Routing

Choose the runtime base by verified requirements:

1. Use `scratch` for a fully static artifact only when it needs no CA certificates, timezone or locale data, user database, shell, package manager, or other operating-system files.
2. Use distroless when a matching runtime exists and the application needs a minimal set of runtime libraries or system data.
3. Use Alpine when the application needs a minimal conventional userspace and is compatible with musl and the available native dependencies.
4. Use Debian slim only for a concrete incompatibility such as glibc, unavailable native packages, or vendor support. Add a concise comment explaining the reason when it is not evident from the copied artifact.

Use trusted official or verified publisher images. Retain a readable exact version tag and pin its digest in production:

```dockerfile
FROM alpine:3.23.3@sha256:... AS runtime
```

Never use `latest`. Never select a base solely because it produces the smallest compressed number; confirm architecture availability, support lifecycle, runtime compatibility, and vulnerability posture.

## Stages and Artifacts

- Declare `# syntax=docker/dockerfile:1` and use BuildKit.
- Give stages lowercase semantic names such as `dependencies`, `build`, `test`, and `runtime`.
- Keep dependency resolution separate from frequently changing source when that improves cache reuse.
- Make test and lint stages buildable targets, but keep them outside the runtime ancestry.
- Copy final artifacts from an exact named stage. Do not copy an entire builder filesystem.
- Never install a compiler in the runtime stage merely to build a native dependency at container startup.
- Avoid broad early `COPY . .`; first copy only inputs needed for dependency resolution, then the minimum source needed for the build.

## Build Inputs and Caching

- Require repository lockfiles and the ecosystem's frozen or immutable install mode when available.
- Use `RUN --mount=type=cache` for package and compiler caches. A cache mount improves build speed but must not be required for correctness.
- Use `RUN --mount=type=secret` or `RUN --mount=type=ssh` for private dependencies and credentials.
- Never store secrets in `ARG`, `ENV`, labels, URLs, copied configuration, or committed `.dockerignore` exceptions.
- Keep build arguments for genuine build-time variation, not runtime configuration or secrets.
- Sort long package lists for reviewability. Install only packages required by that stage.
- Combine package index refresh, installation, and cleanup in one layer where the base package manager requires it. Do not run blanket `apk upgrade` or `apt-get upgrade`.

## Runtime Hardening

- Use non-root execution. Prefer the runtime base's standard non-root identity; otherwise use stable numeric IDs such as `10001:10001`.
- Use `COPY --chown` and narrowly scoped directory creation. Avoid recursive ownership changes over large trees.
- Preserve only permissions the process needs. Do not make application files globally writable.
- Design for `docker run --read-only`; identify exact tmpfs or volume mounts needed for writes.
- Keep one concern per image. Run the application directly as PID 1 and use exec-form startup instructions.
- Add an init only after demonstrating defective signal forwarding or child reaping without it.
- Do not include shells, editors, process monitors, curl, wget, Git, compilers, headers, or package managers in the final stage. When a base supplies unwanted tooling inherently, prefer a stricter base or document the exception.
- Keep root-only setup in build layers. If runtime root is unavoidable, explain the constraint and minimize Linux capabilities and writable filesystem access in the deployment guidance.

## Reproducibility and Supply Chain

- Pin base tags and digests; update them through reviewed automation or deliberate maintenance.
- Verify remote artifacts with a fixed checksum or a trusted signature. Use `ADD --checksum` only with an immutable or versioned HTTPS source; otherwise download and verify in a disposable builder.
- Do not use unverified install scripts piped from the network into a shell.
- Add real OCI labels when known: `org.opencontainers.image.source`, `.revision`, `.version`, and `.licenses`.
- Omit `org.opencontainers.image.created` unless the release process explicitly requires it, because timestamps undermine reproducibility.
- Ensure every selected base and downloaded artifact exists for both `linux/amd64` and `linux/arm64`.
- Prefer deterministic build inputs and honor mechanisms such as `SOURCE_DATE_EPOCH` when the project already supports them.
- Request SBOM and provenance attestations in Buildx publication commands; keep registry pushes outside ordinary local verification unless authorized.

## Instruction Rules

- Prefer `COPY`; use `ADD` only for a feature it uniquely provides, such as verified remote artifacts or intentional local archive extraction.
- Use JSON exec form for `ENTRYPOINT` and `CMD`. Use `ENTRYPOINT` for a fixed executable and `CMD` for default arguments when that interface is useful; otherwise use a single clear `CMD`.
- Set `WORKDIR` explicitly. Use absolute paths.
- Add `EXPOSE` only for a stable documented listening port.
- Omit Dockerfile `HEALTHCHECK` by default. Preserve or add one only when the repository deliberately owns image-level health semantics.
- Keep comments rare and useful: explain compatibility exceptions, non-obvious security decisions, or required workarounds.
- Do not add empty placeholders, speculative arguments, debug targets, or development conveniences to a production Dockerfile.

## `.dockerignore`

Start from repository evidence. Normally exclude:

```text
.git
.env
.env*
!.env.example
**/*.pem
**/*.key
**/id_rsa*
**/credentials.json
**/node_modules
**/.venv
**/target
**/dist
**/build
**/coverage
**/.cache
**/*.log
```

Adapt names to the repository; these are examples, not a block to paste blindly. Re-include a file only when the build demonstrably consumes it. Do not assume `.git`, tests, documentation, or generated directories are unused; version derivation and build steps sometimes require them.

Official references:

- <https://docs.docker.com/build/building/best-practices/>
- <https://docs.docker.com/build/building/multi-stage/>
- <https://docs.docker.com/build/cache/optimize/>
- <https://docs.docker.com/build/building/secrets/>
- <https://docs.docker.com/build/building/multi-platform/>
