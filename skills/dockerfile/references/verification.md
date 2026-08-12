# Local Dockerfile Verification

Set task-specific variables to the actual file, context, image, and targets. Do not assume the repository root is the build context.

```bash
dockerfile_path=Dockerfile
build_context=.
runtime_image=local/dockerfile-qa:validation
```

## Static Checks

Run Docker's checks as an error-producing gate:

```bash
docker build --check \
  --build-arg 'BUILDKIT_DOCKERFILE_CHECK=error=true' \
  --file "$dockerfile_path" "$build_context"
```

Run Hadolint when installed:

```bash
hadolint "$dockerfile_path"
```

Do not suppress a rule merely to get green. Document a narrowly justified exception next to the affected instruction only when the rule conflicts with a verified requirement.

## Build Every Target

List declared targets and build each meaningful terminal stage, including tests:

```bash
docker buildx build --call=targets \
  --file "$dockerfile_path" "$build_context"

docker buildx build --target test \
  --file "$dockerfile_path" "$build_context"
```

Omit target commands for stages that do not exist. Pass the same non-secret build arguments used by the real build. Supply credentials only with `--secret` or `--ssh`.

## Verify Both Platforms

Build the complete runtime graph for both required platforms without pushing:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --output type=cacheonly \
  --file "$dockerfile_path" "$build_context"
```

If the active builder cannot execute or cross-compile a required stage, configure an appropriate builder or report that platform as unverified. Never claim multi-platform support from Dockerfile inspection alone.

Build and load the native platform for runtime QA:

```bash
docker buildx build --load \
  --tag "$runtime_image" \
  --file "$dockerfile_path" "$build_context"
```

## Exercise the Image

Run the application's real interface: make an HTTP request, invoke the CLI, process a representative input, or exercise the service protocol. A successful build is not sufficient.

Start with a read-only root filesystem and only the proven writable mounts:

```bash
docker run --detach --rm \
  --name dockerfile-runtime-qa \
  --read-only \
  --tmpfs /tmp:rw,nosuid,nodev,noexec \
  "$runtime_image"
```

Adapt ports, arguments, environment, and writable mounts to the application. Do not inject production credentials. If the application does not use `/tmp`, omit that mount.

Verify the configured user without assuming the image has a shell:

```bash
docker image inspect "$runtime_image" \
  --format '{{json .Config.User}}'
```

Exercise startup, readiness, normal work, failure behavior, and graceful shutdown. Use `docker stop` and inspect logs and exit status; do not merely kill the process.

## Inspect the Artifact

Record size, labels, startup configuration, and history:

```bash
docker image inspect "$runtime_image" \
  --format 'size={{.Size}} user={{json .Config.User}} entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}} labels={{json .Config.Labels}}'
docker history --no-trunc "$runtime_image"
```

Use an available image inspection tool to confirm the final filesystem contains no source, secrets, caches, package indexes, compiler toolchain, VCS/debugging utilities, or unexpected writable permissions. Do not rely on layer count alone; modern Docker instructions do not necessarily create extra filesystem layers.

Run a vulnerability scan when available:

```bash
trivy image "$runtime_image"
```

Treat unresolved critical or high findings as blockers unless they are pre-existing, unreachable, or unfixable and explicitly documented with evidence. Never hide scanner output with a broad ignore.

## Publication Evidence

In an authorized publication workflow, request both required platforms plus SBOM and provenance attestations:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --sbom=true \
  --provenance=mode=max \
  --push \
  --tag REGISTRY/IMAGE:TAG \
  --file "$dockerfile_path" "$build_context"
```

Never run the push command during ordinary validation without explicit authorization.

Official references:

- <https://docs.docker.com/build/checks/>
- <https://docs.docker.com/build/metadata/attestations/>
