# Instruments

Each command writes into `$H`. Replace `<src>` with the source directories.
Every hit is a lead for step 5 of the skill, never a finding by itself.

## Every project

```
go run github.com/zricethezav/gitleaks/v8@latest git --no-banner --redact -f csv -r "$H/secrets.csv" . 2>/dev/null
go run github.com/google/osv-scanner/v2/cmd/osv-scanner@latest scan source -r -f markdown . > "$H/osv.md" 2>/dev/null
docker run --rm -v "$PWD:/src:ro" aquasec/trivy:latest fs --quiet --scanners misconfig,secret,license --format table /src > "$H/trivy.txt" 2>&1
git ls-files -ci --exclude-standard > "$H/tracked-ignored.txt"
git ls-files | grep -iE '(^|/)\.env(\.|$)|\.pem$|\.key$|id_rsa|\.p12$|\.pfx$' > "$H/secret-files.txt"
git grep -nE 'DEBUG *= *(True|true|1)|NODE_ENV.*development|APP_DEBUG=true|FLASK_DEBUG|\bdebug: *true' > "$H/debug-defaults.txt"
grep -rnE "Access-Control-Allow-Origin.*\*|origin: *['\"]\*|allow_origins=\[['\"]\*|AllowAllOrigins|CORS_ALLOW_ALL" <src> > "$H/cors.txt"
grep -rnE 'verify=False|rejectUnauthorized: *false|InsecureSkipVerify: *true|NODE_TLS_REJECT_UNAUTHORIZED|CURLOPT_SSL_VERIFYPEER.*(false|0)' <src> > "$H/tls-off.txt"
grep -rnE "(secret|password|passwd|api_?key|token)[\"']? *[:=] *[\"'][^\"'\$\{]{6,}" -i <src> | cut -d: -f1,2 > "$H/hardcoded-creds.txt"
grep -rnE 'SIGTERM|SIGINT|signal\.Notify|NotifyContext|process\.on\(|add_signal_handler|graceful' <src> > "$H/signals.txt"
grep -rnEi "['\"/](health|healthz|ready|readyz|live|livez|ping|status)['\"]" <src> > "$H/health.txt"
grep -rnE '(log|logger|console)\.[a-z]+\(.*(password|token|secret|authorization|cookie|ssn|card)' -i <src> > "$H/log-leaks.txt"
git grep -nEi 'sentry|bugsnag|rollbar|honeybadger|opentelemetry|prometheus|datadog|newrelic|statsd' > "$H/observability.txt"
grep -rnEi 'rate.?limit|throttle|slowapi|express-rate-limit|limiter' <src> > "$H/rate-limit.txt"
find . -path ./node_modules -prune -o \( -ipath '*migrat*' -o -name '*.sql' \) -type f -print > "$H/migrations.txt"
tr '\n' '\0' < "$H/migrations.txt" | xargs -0 grep -HnEi 'drop (table|column)|rename (table|column)|alter column .* not null|truncate' > "$H/destructive-migrations.txt" 2>/dev/null
grep -rnE 'continue-on-error: *true|allow_failure: *true|\|\| *true' .github .gitlab-ci.yml .circleci > "$H/ci-cannot-fail.txt" 2>/dev/null
git ls-files .nvmrc .node-version .python-version .tool-versions 'rust-toolchain*' go.mod runtime.txt > "$H/runtime-pins.txt"
```

- **Runtime end of life**: take the runtime versions from `runtime-pins.txt`,
  the Dockerfile `FROM` lines and the manifest, and check each one against
  `https://endoflife.date/api/<product>.json`. This sends only the product
  name.
- **Secrets**: `--redact` and `cut` keep values out of the files, and
  `git grep` skips untracked files such as a local `.env`. A `.env` that is
  tracked counts as a secret even when it holds only placeholders today.
- **Health and signals**: an empty file is a lead that is only worth anything
  after the deploy target is known. A platform health check or a framework
  that handles SIGTERM itself refutes it.
- **trivy** needs Docker. Without it, `go run github.com/aquasecurity/trivy/cmd/trivy@latest`
  works but builds for minutes. Report which one ran.

## Outbound calls with no timeout

```
grep -rnE 'requests\.(get|post|put|patch|delete|request)\(' --include='*.py' <src> | grep -v 'timeout=' > "$H/py-no-timeout.txt"
grep -rnE 'httpx\.(Client|AsyncClient)\(\)|urlopen\(' --include='*.py' <src> > "$H/py-client-default.txt"
grep -rnE '\bfetch\(' --include='*.ts' --include='*.js' --include='*.tsx' <src> | grep -v 'signal' > "$H/js-fetch.txt"
grep -rnE 'axios\.create\(\)|new (Pool|Client)\(\)' --include='*.ts' --include='*.js' <src> > "$H/js-client-default.txt"
grep -rnE 'http\.(Get|Post|Head)\(|&http\.Client\{\}|http\.DefaultClient' --include='*.go' <src> > "$H/go-no-timeout.txt"
grep -rnE 'reqwest::(get|Client::new)\(' --include='*.rs' <src> > "$H/rs-no-timeout.txt"
```

Each hit holds until a global setting refutes it: a configured client, an
`axios` default, an undici dispatcher, or an `http.Client` built in one place
with `Timeout`.

## Unbounded resources

```
grep -rnE '\.(all|find|findMany|select)\(\)|\.objects\.all\(\)|SELECT \* FROM [a-z_]+ *;?["'"'"']' <src> > "$H/unbounded-queries.txt"
grep -rnE 'express\.json\(\)|bodyParser\.json\(\)|MaxBytesReader|client_max_body_size|DATA_UPLOAD_MAX' <src> > "$H/body-limits.txt"
grep -rnE 'new Map\(\)|= *\{\} *$|lru|cachetools|@cache\b|@lru_cache\(maxsize=None' <src> > "$H/caches.txt"
```

These are noisy. Keep only the module-level caches and queries on tables that
grow with users or time.

## Dockerfile

```
docker run --rm -i hadolint/hadolint < Dockerfile > "$H/hadolint.txt" 2>&1
git grep -niE -e '^FROM .*:latest' -e '^FROM +(--platform=[^ ]+ +)?[^:@ ]+( +AS +.*)?$' -- 'Dockerfile*' > "$H/unpinned-base.txt"
git grep -nE '^USER ' -- 'Dockerfile*' > "$H/user.txt"
git grep -nE '^HEALTHCHECK|^STOPSIGNAL|^(CMD|ENTRYPOINT) .*(sh"?,? *"?-c|npm)' -- 'Dockerfile*' > "$H/lifecycle.txt"
```

- No `USER`, or `USER root` in the final stage, means the container runs as
  root.
- npm in `CMD` or `ENTRYPOINT`, and `sh -c`, put npm or a shell in PID 1,
  which swallows SIGTERM. The shutdown timing in step 3 proves it.
- A `FROM` hit that names an earlier stage is not an unpinned base.
- `docker build` from the scratch copy is step 3's clean build. Afterwards,
  `docker image inspect` gives the size, user and health check of what was
  built.

## Kubernetes, Helm, Compose

```
docker run --rm -v "$PWD:/src:ro" aquasec/trivy:latest config --quiet /src > "$H/trivy-config.txt" 2>&1
grep -rnE 'resources:|limits:|requests:|readinessProbe|livenessProbe|startupProbe|terminationGracePeriodSeconds|replicas:|PodDisruptionBudget' --include='*.yaml' --include='*.yml' . > "$H/k8s-lifecycle.txt"
```

A Deployment with no limits, no readiness probe, or `replicas: 1` for a
service that must stay up is a risk.

## Terraform

`trivy config` above covers it. In addition, grep for state in the repository
(`*.tfstate`, which counts as a secret) and for `prevent_destroy` on the
databases and buckets that hold data.

## GitHub Actions

```
uvx zizmor --format plain .github/workflows > "$H/zizmor.txt" 2>&1
go run github.com/rhysd/actionlint/cmd/actionlint@latest > "$H/actionlint.txt" 2>&1
```

Check that a test job gates the deploy job (`needs:`). A deploy that runs
without the tests passing is a blocker. Fixes belong to the `github-actions`
skill.

## Language extras

- **Go**: `go run golang.org/x/vuln/cmd/govulncheck@latest ./... > "$H/govulncheck.txt"`.
  It reports only the vulnerabilities the code reaches, which settles
  "reached?" for step 5.
- **JS/TS**: `npm audit --omit=dev --package-lock-only > "$H/audit.txt"`.
  Check `package.json` `engines` against the Dockerfile's Node version.
- **Python**: `uvx pip-audit -r requirements.txt > "$H/pip-audit.txt"` or the
  project's lockfile. A bare `print` in server code is a sign of missing
  logging.
- **Rust**: `cargo audit`, if installed. Also look for `unwrap()` in request
  handlers, which panics on bad input.
- **PHP**: `composer audit --locked`. Check that `display_errors` and
  `APP_DEBUG` are off in the production config.
