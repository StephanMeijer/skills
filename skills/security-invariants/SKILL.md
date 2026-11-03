---
name: security-invariants
description: Design, audit, test, and repair security invariants across authentication, authorization, tokens, sessions, cookies, federation, proxy trust, registration, rate limiting, and other trust boundaries. Use for security reviews, security-sensitive features, pre-release audits, vulnerability remediation, or when equivalent request paths must enforce the same policy. Do not use for dependency-only vulnerability scanning.
license: MIT
---

# Security Invariants

Find the security failures happy-path review misses and leave regression tests that make the same
class of failure hard to reintroduce.

## Choose the Job

- **Prevent**: before or during a security-sensitive change, define the invariant, enumerate every
  equivalent path, and design negative tests. Implement only when the user asked for implementation.
- **Audit**: inspect and test the current system, read-only unless the user also asks for fixes.
- **Remediate**: reproduce the reported failure, add a regression test that fails for the right
  reason, make the smallest centralized fix, and verify every sibling path.

Do not turn an ordinary code change into a whole-repository audit. Do not publish vulnerability
details, contact maintainers, or mutate external systems without separate authorization.

## Establish the Security Contract

Read the repository guidance, security policy, architecture, tests, deployment examples, and relevant
configuration. Identify:

- assets and operations worth protecting;
- attacker-controlled values and actors with partial privileges;
- trust boundaries, including proxies and external identity providers;
- security decisions, the code that owns each decision, and failure behavior;
- all producers, consumers, refreshers, exchangers, and revokers of protected artifacts.

Record assumptions as assumptions: a documented deployment example shows intended operation, not how
every deployment behaves.

## Build an Invariant Ledger

For each security decision, write one falsifiable invariant that constrains observable behavior, for
example: an issued token never has more authority or a longer lifetime than any input authority
permits. Name its enforcement owner.

When several routes, grants, protocols, roles, configurations, or artifact types must obey the same
rule, build the path matrix in [references/invariant-matrix.md](references/invariant-matrix.md). An
uninspected cell is an evidence gap, not a pass.

Prefer one shared enforcement point every path crosses; duplicated checks are a prompt to prove
parity, not evidence of it.

## Attack the Boundaries

Read only the relevant sections of
[references/attack-surfaces.md](references/attack-surfaces.md), then test individual boundaries and
composed sequences. Look for:

- input reaching fatal operations or stricter downstream parsers;
- substring, prefix, suffix, or uncanonicalized comparisons standing in for structured membership;
- authority, audience, or lifetime copied rather than intersected;
- signed or encrypted artifacts replayable under another name or purpose;
- external identities interpreted in a local namespace;
- forwarded metadata trusted without a complete hop model;
- counters, locks, and revocations defeated by success, retry, concurrency, replay, or reordering.

Search and static-analysis hits are candidates. Report a vulnerability only after tracing a reachable
path and establishing attacker control, the violated invariant, and the consequence.

## Prove and Repair

Use the cheapest evidence that answers the question, then escalate for high-impact claims:

1. static path trace;
2. focused unit or property test against real production code;
3. integration test at the actual protocol or process boundary;
4. safe live reproduction in an isolated environment.

For remediation, first lock the failure with a deterministic test that fails on the vulnerable code,
then fix the invariant at its owner, not each observed caller, and re-run the complete path matrix:
negative cases, alternate configuration, expiration, replay, and the adversarial event sequences
relevant to the change. A finding is resolved when the regression test, the centralized invariant,
and sibling-path verification exist, not when the suspicious line disappears.

## Report

Read [references/evidence-and-severity.md](references/evidence-and-severity.md) before producing an
audit or vulnerability report. Separate observed behavior from inference, state prerequisites, and
label code-only claims plainly. Combine variants with one root cause; keep independent security
boundaries separate.

End with:

- the invariant ledger and path matrix;
- findings ordered by consequence and exploitability;
- exact evidence and reproduction status;
- unverified paths and deployment assumptions;
- the smallest credible repair and required regression coverage.
