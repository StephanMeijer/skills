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

Use trusted official or verified publisher images. Retain a meaningful publisher tag and pin the OCI image-index or manifest-list digest that contains both required platforms:

```dockerfile
FROM alpine:3.23.3@sha256:... AS runtime
```

Verify the pinned digest and its platforms before use:

```bash
docker buildx imagetools inspect IMAGE:TAG
docker buildx imagetools inspect IMAGE@sha256:INDEX_DIGEST
```

Do not pin a platform-specific manifest digest in a Dockerfile that must build for both architectures. Not every publisher offers patch-version tags; distroless commonly uses tags that communicate runtime and user variants instead. Never use `latest`. Never select a base solely because it produces the smallest compressed number; confirm architecture availability, support lifecycle, runtime compatibility, and vulnerability posture.

## Stages and Artifacts

- Declare `# syntax=docker/dockerfile:1` and use BuildKit.
- Give stages lowercase semantic names such as `dependencies`, `build`, `test`, and `runtime`.
- Keep dependency resolution separate from frequently changing source when that improves cache reuse.
- Add buildable test or lint stages only when the repository already uses that design or the tests must exercise build-stage artifacts. Keep them outside the runtime ancestry.
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
- Do not install shells, editors, process monitors, curl, wget, Git, compilers, headers, or package managers in the final stage. Prefer distroless or `scratch` when these tools must be absent. Alpine inherently includes BusyBox and `apk`; accept and report that tradeoff only when Alpine's userspace is actually required.
- Keep root-only setup in build layers. If runtime root is unavoidable, explain the constraint and report the minimum required capabilities and writable paths without silently expanding into deployment-file edits.

## Reproducibility and Supply Chain

- Pin base tags and digests; update them through reviewed automation or deliberate maintenance.
- Verify remote artifacts with a fixed checksum or a trusted signature. Use `ADD --checksum` only with an immutable or versioned HTTPS source; otherwise download and verify in a disposable builder.
- Do not use unverified install scripts piped from the network into a shell.
- Add real OCI labels when known: `org.opencontainers.image.source`, `.revision`, `.version`, and `.licenses`.
- Omit `org.opencontainers.image.created` unless the release process explicitly requires it, because timestamps undermine reproducibility.
- Ensure every selected base and downloaded artifact exists for both `linux/amd64` and `linux/arm64`.
- Choose a real multi-platform strategy: native builder nodes, QEMU emulation, or cross-compilation. Use `FROM --platform=$BUILDPLATFORM` only when the toolchain can cross-compile for `$TARGETOS/$TARGETARCH`; otherwise build target-platform stages on native nodes or with registered emulation. Never execute a target-architecture binary on the build platform without the required emulation.
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

Derive `.dockerignore` from the actual build context; never paste a static language or framework template.

1. Enumerate tracked and untracked paths under the selected build context.
2. Trace every `COPY`, `ADD`, bind mount, build script, versioning step, and test target to identify inputs that must remain available.
3. Exclude everything else that is local-only, reproducibly generated, irrelevant to the build, or sensitive. Pay particular attention to repository metadata, credentials, private keys, environment files, local dependency trees, caches, logs, coverage, editor state, and host build outputs, but match the repository's real paths rather than broad guessed extensions.
4. Preserve required workspace manifests, generated sources, tests, documentation, public certificates, and VCS metadata when a verified build step consumes them.
5. When multiple Dockerfiles use materially different contexts, consider Dockerfile-specific ignore files instead of weakening one global file.
6. Re-run build checks and every target after changing ignore rules. A smaller context does not justify a missing build input.

Use `.gitignore` only as evidence; do not copy it wholesale. Git-ignored files may be required build inputs, while tracked files may still be irrelevant or unsafe to send to the builder.

Official references:

- <https://docs.docker.com/build/building/best-practices/>
- <https://docs.docker.com/build/building/multi-stage/>
- <https://docs.docker.com/build/cache/optimize/>
- <https://docs.docker.com/build/building/secrets/>
- <https://docs.docker.com/build/building/multi-platform/>
