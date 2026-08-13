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

For a one-shot CLI, run it in the foreground with a read-only root filesystem and only proven writable mounts:

```bash
docker run --rm \
  --read-only \
  --tmpfs /tmp:rw,nosuid,nodev,noexec \
  "$runtime_image"
```

For a service, retain the stopped container until its logs and exit state have been inspected:

```bash
container_name="dockerfile-runtime-qa-$$"
docker run --detach \
  --name "$container_name" \
  --read-only \
  --tmpfs /tmp:rw,nosuid,nodev,noexec \
  "$runtime_image"

# Exercise the real readiness and service interface here.

docker stop --time 10 "$container_name"
docker logs "$container_name"
test "$(docker inspect --format '{{.State.ExitCode}}' "$container_name")" -eq 0
docker rm "$container_name"
```

The readiness and interface step is mandatory; replace the marker with the repository's real probe before running the scenario. Adapt ports, arguments, environment, and writable mounts to the application. Do not inject production credentials. If the application does not use `/tmp`, omit that mount. On failure, inspect logs and state before removing the container.

Verify the configured user without assuming the image has a shell. Unless a root exception is documented, make root-equivalent values fail:

```bash
image_user="$(docker image inspect "$runtime_image" --format '{{.Config.User}}')"
case "$image_user" in
  ""|0|0:*|root|root:*)
    printf 'runtime image is configured as root: %s\n' "$image_user" >&2
    exit 1
    ;;
esac
```

A name passes that check without proving anything, so resolve it against the image's own user database. Skip the lookup for a numeric ID, which the case above already proved non-zero:

```bash
user_name="${image_user%%:*}"
case "$user_name" in
  [0-9]*) ;;
  *)
    inspection_container="$(docker create "$runtime_image")"
    passwd_uid="$(docker cp "$inspection_container:/etc/passwd" - 2>/dev/null \
      | tar -xO 2>/dev/null \
      | awk -F: -v name="$user_name" '$1 == name { print $3 }')"
    docker rm "$inspection_container" >/dev/null

    if test -z "$passwd_uid"; then
      printf 'runtime image user %s does not resolve; the container cannot start\n' "$user_name" >&2
      exit 1
    fi
    if test "$passwd_uid" -eq 0; then
      printf 'runtime image user %s resolves to UID 0\n' "$user_name" >&2
      exit 1
    fi
    ;;
esac
```

An unresolvable name is a failure rather than an unverified check, because the runtime rejects it too: `docker run` on such an image fails with `unable to find user NAME: no matching entries in passwd file`. Copying the file out of a created container keeps the check working on an image with no shell, and reaching a `scratch` image with no user database at all correctly reports the name as unresolvable.

Exercise startup, readiness, normal work, failure behavior, and graceful shutdown. Use `docker stop` and inspect logs and exit status; do not merely kill the process.

## Inspect the Artifact

Record size, labels, startup configuration, and history:

```bash
docker image inspect "$runtime_image" \
  --format 'size={{.Size}} user={{json .Config.User}} entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}} labels={{json .Config.Labels}}'
docker history --no-trunc "$runtime_image"
```

Inventory packages and the final filesystem with Syft when installed:

```bash
syft "$runtime_image"
```

Without Syft, export a created container and inspect the complete path list without requiring a shell in the image:

```bash
inspection_container="$(docker create "$runtime_image")"
docker export "$inspection_container" | tar -tf - | sort
docker rm "$inspection_container"
```

Compare the inventory with the selected base and intended artifacts. Fail the review when it reveals copied source, secrets, caches, package indexes, an unexpected compiler toolchain, or unjustified VCS/debugging utilities. Alpine's built-in BusyBox and `apk` are base-image contents, not proof that the Dockerfile installed debugging tools; record that accepted base tradeoff. Do not rely on layer count alone because modern Docker instructions do not necessarily create filesystem layers.

Run a vulnerability and secret scan when Trivy is available. Make fixable high or critical findings fail the command:

```bash
trivy image \
  --scanners vuln,secret \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --exit-code 1 \
  "$runtime_image"
```

Run a second reporting scan without `--ignore-unfixed` when the first scan omitted findings. Report unfixed high or critical findings and assess them explicitly; do not silently classify “unfixed” as safe. Never hide scanner output with a broad ignore.

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
