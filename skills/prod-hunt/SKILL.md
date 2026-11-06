---
name: prod-hunt
description: Production-readiness hunt. Sweep a repository for everything that stops it from going live and staying up, and prove each find. Covers a build that only works on one machine, committed secrets, reachable vulnerable dependencies, insecure defaults, destructive migrations, outbound calls with no timeout, no graceful shutdown, no health check, missing logs, metrics and error reporting, unbounded resources, a root container, an end-of-life runtime, and no rollback or runbook. Tools find the mechanical blockers, running the app in a sandbox proves the rest, and the reply is a verdict plus ranked findings a developer can pick up cold. Use when asked whether a project is production-ready, for a go-live or launch-readiness check, a pre-release audit, or what to fix before shipping. Reports in conversation and does not fix. Repository only; never touches live systems.
license: MIT
---

# Production-readiness hunt

"Is it production-ready?" usually gets a checklist ticked from memory. This hunt
answers with evidence: what will break, leak or page someone once real traffic,
real data and real failures arrive, and how anyone can see that today.

**A blocker without evidence is an opinion.** "Logging is weak" is worthless.
"`src/pay.ts:42` calls the payment API with `fetch` and no `signal`; when the
provider hangs, every worker hangs with it" is a finding.

**Name the production failure.** Every finding ends in an outage, data loss, a
breach, a silent failure, or a page at 3 a.m. Style, taste and code quality are
not this hunt; those belong to `suck-hunt`.

## 1. Ground rules

- Default to the whole repository. Record `git rev-parse --short HEAD` and
  whether the tree is dirty. If it is dirty, the report says its line numbers
  are for HEAD plus uncommitted changes.
- **Learn the deploy target first**: container, Kubernetes, serverless, PaaS
  (`Procfile`, `fly.toml`, `render.yaml`), VM, or library. Read the Dockerfile,
  IaC, CI deploy jobs and README. Many checks depend on the target: a library
  needs no health endpoint, and a platform may already provide TLS, restarts or
  health checks. Say which target you assumed.
- **Use one hunt directory for everything.** Run
  `H=$(mktemp -d) && git status --porcelain --ignored > "$H/before" && echo "$H"`.
  Every tool writes to a file in `$H`, never to the conversation. Count the
  output first, then grep what you need. Subagents get the same path.
- **The repository is read-only.** Experiments run in a scratch copy:
  `mkdir "$H/copy" && git archive HEAD | tar -x -C "$H/copy"`. At the end, diff a
  fresh `git status --porcelain --ignored` against `$H/before` and delete only
  the new paths. Keep `$H`.
- **Running project code is not free.** Read what will run first. Run it only
  when it is self-contained: nothing that migrates a database the run did not
  create, calls a real external service, sends mail or deploys. Containers run
  with `--network none` unless they truly need a local dependency, and you
  start that dependency yourself in `$H`. Stop every run after ten minutes.
  Anything that cannot run safely is reported as "not run", with the reason.
- **Never touch live systems.** No requests to staging or production URLs, no
  cloud CLIs against real accounts, no reading of real `.env` values. Secrets
  never reach the conversation: name the file, the commit and the kind of
  secret, never the value.
- Fetched tools are pinned where the commands pin them. The report gives the
  version of every tool the project does not pin. osv-scanner and trivy send
  package names and versions, not code, to public databases. If the dependency
  list is confidential, ask first.

## 2. Let the instruments speak first

Run the commands in [references/instruments.md](references/instruments.md):
the every-project set, then the set for each language and each deploy artifact
the project has (Dockerfile, Kubernetes or Helm, Terraform, GitHub Actions).
Tool output is a lead, not a finding. Everything goes through step 5.

## 3. Prove it by running

Static reading guesses; a run proves. Where step 1 allows it:

- **Clean build.** Build the scratch copy exactly as CI or the Dockerfile
  does. If it fails, the build depends on something that exists only on the
  author's machine: an untracked file, a global tool, or a missing lockfile.
  That is a blocker.
- **Start without config.** Start the app with no environment variables. Good:
  it exits non-zero within seconds and names the missing variable. Bad: it
  starts with a default secret, `DEBUG` on, or an in-memory store, or it
  crashes later on the first request.
- **Shutdown.** Start the container, then time
  `docker stop -t 30 <id>`. Close to 30 seconds means SIGTERM is ignored:
  in-flight requests die and every deploy waits out the timeout. Also check
  `docker inspect` for the user (root?) and confirm the health check command
  actually succeeds.
- **Health.** If a health or readiness route exists, call it inside the
  sandbox. A route that returns 200 while the database is down, or one that
  needs auth, does not protect anything.
- **Migrations.** Run them up and down against a throwaway database in `$H`
  if the tooling supports it. Read every migration for drops, renames and
  `NOT NULL` without a default on existing tables. Those break the old code
  still running during a rolling deploy.

## 4. Hunt

Read where production failures start: entry points, request handlers,
outbound clients, job workers, config loading, the Dockerfile and deploy
config. Then read the files the tools pointed at. On a large codebase, fan
out by area, one subagent per service or directory. Each subagent hunts every
category below and receives `$H` and these rules.

**Usually blockers** (do not go live with these):

- **Won't build or start**: a clean build fails, the entry point crashes, or
  the runtime version is unpinned or end-of-life.
- **Secrets**: a key, token or password in the tree or in history. Rotating it
  is step one of the fix; deleting it from history does not un-leak it.
- **Vulnerable dependencies**: a known vulnerability with a fix available, in
  code that ships and is reached.
- **Insecure defaults**: debug or stack traces on in production config, auth
  off by default, CORS `*` with credentials, TLS verification off, admin or
  debug endpoints exposed, a default password, a hard-coded session secret.
- **Data loss**: destructive or irreversible migrations, no transactions
  around multi-step writes, deletes with no backup or soft-delete on data
  that matters, and a stateful service with no word about backups anywhere.
- **No safety net**: no tests on money, auth or data paths; CI that does not
  run them, or cannot fail (`continue-on-error`, `|| true`).

**Usually risks** (these become the first month of incidents):

- **Failure handling**: outbound HTTP, database or queue calls with no
  timeout; retries with no backoff or limit; errors swallowed; one failing
  dependency taking the whole process down; unhandled promise rejections or
  panics in workers.
- **Unbounded resources**: queries without pagination or limit, caches and
  queues that only grow, request bodies with no size limit, no connection
  pool limit, and no rate limit on public or login endpoints. State the size
  each one reaches in real use.
- **Lifecycle**: no SIGTERM handling, no readiness separate from liveness,
  startup that blocks on a dependency forever, work lost when a job is
  killed mid-flight.
- **Observability**: unstructured or absent logs, no request or correlation
  IDs, no metrics, no error reporting, log levels fixed at debug, and secrets,
  tokens or PII written to logs. The test is whether someone on call could
  tell what broke from the logs alone.
- **Container and runtime**: running as root, an unpinned or `:latest` base
  image, no resource limits in the deploy manifests, a writable root
  filesystem without need, dev dependencies or build tools in the final image.
- **Config**: config read lazily instead of validated at startup,
  environment-specific values hard-coded, and dev and prod sharing defaults
  that only make sense in dev.

**Usually ops gaps** (fine on day one, painful on day thirty):

- **Deploy and rollback**: no documented deploy, no way to roll back, and
  schema changes that make rollback impossible. Releases are not versioned,
  so nobody can say what is running.
- **Runbook**: no list of required environment variables, ports, dependencies
  or what to do when a known thing breaks.
- **Licensing**: copyleft dependencies in proprietary software that ships, or
  no LICENSE where one is expected.

Hand off rather than duplicate. Deep security work goes to
`security-invariants` or `red-blue`; container fixes to `dockerfile`; CI fixes
to `github-actions`; whether the setup docs work to `cold-start`; code quality
to `suck-hunt`; frontend performance to `crash-test`. One finding that points
at the right skill beats a shallow version of its job.

## 5. Try to kill every find

Hunters overclaim. Try to prove each finding wrong. Where subagents are
available, give one refuter each area's batch. It answers "holds" or "refuted,
because …" for every finding.

- **No timeout**: is it set on the client globally, by a wrapper, or by the
  platform (for example a load balancer or a Lambda timeout)? A platform
  timeout still leaves the worker hanging, so say which one applies.
- **No health check or TLS or restarts**: does the platform provide it? Read
  the deploy config, not just the code.
- **Insecure default**: is it the production value, or is it overridden in
  prod config or deploy manifests?
- **Vulnerable dependency**: does shipped code reach it, or is it dev-only?
- **Unbounded**: where does the input come from, and how big does it get?
  If it stays small, drop the finding.
- **Secret**: is it a real credential, or a documented test fixture?
- **Migration**: does the project deploy in a way where old and new code
  run at the same time? If deploys are stop-the-world, drop the rolling
  deploy argument but keep data loss.

Write every survivor down the moment it survives, one file per finding:
`$H/findings/<band>-<category>-<slug>.md`, in the format from step 6.
Subagents prefix the name with their area: `<area>--<band>-<category>-<slug>.md`.

## 6. Report in the conversation

Write nothing into the repository. When every subagent is done, the lead
merges duplicate findings, numbers the files in report order, and writes
`$H/findings/INDEX.md` with one row per finding.

**Header**, short:

- the path to `$H/findings/` and the total number of findings;
- the commit, plus "+ uncommitted changes" if the tree is dirty;
- the deploy target assumed;
- what ran, with the version of each fetched tool, and what did not run and
  why;
- what was read, for example: "read 41 of 230 files, entry points and
  clients first; `admin/` not read".

**Verdict**, one line: `Not ready: 3 blockers, 7 risks.` or
`Ready, with 4 risks to fix in the first month.` A project with any blocker
is never "ready".

**Table** of up to 25 findings. Order them blocker, then risk, then ops gap,
and within each band by impact. Then write one line counting the rest, with a
pointer to `INDEX.md`.

```
| #  | Band    | Category | Where                     | Fix                          | Effort | Fix risk |
|----|---------|----------|---------------------------|------------------------------|--------|----------|
| 1  | blocker | secrets  | config/prod.yml:12 (a1b2) | rotate key, load from env    | S      | medium   |
| 2  | risk    | failure  | src/pay.ts:42 charge()    | 5s timeout via AbortSignal   | S      | low      |
```

A category's band moves when the evidence says so. A risk becomes a blocker
when it is on money, auth or data, or on the hot path of every request. A
blocker becomes a risk when the platform already mitigates it, and the finding
says how.

**Full write-ups for the first ten.** Each must stand on its own:

```
### 2 · Add a timeout to the payment call — risk · failure handling · effort S · fix risk low

**Where:** `src/pay.ts:42` `charge()` (at HEAD a1b2c3d); callers: `src/checkout.ts:88`
**What fails in production:** the provider hangs, so every checkout worker
blocks until the pod is killed; checkout is down until the provider recovers.
**Evidence:** `grep -n 'fetch(' src/pay.ts` → `42: const res = await fetch(url, { method: 'POST', body })`;
no `signal`, and no global dispatcher timeout in `src/http.ts`.
**Fix:**
1. Pass `signal: AbortSignal.timeout(5000)` in `charge()`.
2. Map the abort error to the existing `PaymentUnavailable` error.
**Done when:** a test with a server that never answers fails `charge()` within
6 seconds with `PaymentUnavailable`.
**Watch out:** a retry above `charge()` must not double-charge; check that it
sends an idempotency key first.
**Depends on:** —
```

The fields are the same as in `suck-hunt`. **Where** gives file:line and the
symbol. **Evidence** is pasted output or the exact command, not a paraphrase.
**Fix** gives numbered steps naming files and functions. **Done when** is a
check that fails now and passes after. **Effort** is S (under an hour) or M
(up to a day); anything bigger is a project, so give its first S-sized step.
**Fix risk** is low, medium or high; medium and high get **Watch out**.

End with the single change that removes the most production risk for the
least work. Then offer to fix one finding or one band at a time, so each change
can be reviewed on its own.
