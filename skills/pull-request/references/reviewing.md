# Reviewing a Pull Request

Produce actionable findings mapped to the PR diff. A plain review request returns a local draft; publish only when the user explicitly authorizes it.

## Review the Change

1. Surface failed, cancelled, pending, skipped, stale, unavailable, or missing CI before review findings. Place it at the top of the report rather than allowing a green subset to obscure it.
2. Read the complete PR diff and every changed file relevant to a potential finding.
3. Read repository guidance such as `AGENTS.md`, coding standards, and test instructions.
4. Inspect nearby callers, tests, schemas, and error paths before asserting a defect.
5. Prioritize correctness, security, data loss, behavioral regressions, and missing boundary tests. Avoid style-only findings unless they violate an explicit project rule.
6. State each finding with a concrete path and line, impact, triggering scenario, and smallest credible fix.
7. Anchor each line-specific finding to the narrowest relevant changed line in the reviewed diff. Record the provider's new/old side information and the reviewed head commit, not just a local source line.
8. Mark cross-cutting findings, or findings outside a changed diff hunk, as overall-review findings instead of forcing an unrelated inline anchor.
9. Avoid duplicating a concern already covered by an unresolved review thread.
10. If there are no findings, say so and identify any tests or runtime paths that remain unverified.

## Review Security-Sensitive Changes

Use this branch when a diff changes authentication, authorization, token or session issuance, cookies,
identity mapping, federation, proxy-derived metadata, registration, redirects, rate limiting, replay
defenses, or cryptographic artifact handling.

1. State the security invariant the change relies on and identify the shared code that owns it.
2. Find every sibling producer, consumer, refresh, exchange, revocation, and alternate protocol path,
   including unchanged files. Compare their enforcement rather than reviewing the changed handler in
   isolation.
3. Build a compact path-by-invariant matrix. Treat an uninspected applicable cell as an evidence gap,
   not a pass.
4. Verify structured values use exact parsed membership and qualified namespaces. Check that derived
   authority and expiration are the intersection of every principal, client, resource, request, and
   server limit.
5. Verify signed or encrypted artifacts are bound to purpose and consumer, forwarded metadata has a
   complete trusted-hop model, and attacker-controlled input cannot reach a fatal operation or a
   stricter downstream parser unhandled.
6. Review adversarial sequences, not only single requests: failure followed by success, reset, retry,
   replay, expiry, revocation, and concurrency where relevant.
7. Require deterministic negative tests across every applicable sibling path. A happy-path test or a
   passing pipeline is not evidence that the invariant holds.

If the available diff, repository, or environment cannot establish a high-impact claim, report the
gap and the smallest discriminating test. Do not return an unqualified clean security verdict. Put a
verified cross-cutting defect that cannot be anchored to the diff in the overall review with its
exact path and trigger.

## Draft and Submit Fresh Findings

Write one actionable defect per inline comment. Keep each comment self-contained: explain the impact and triggering scenario, then give the smallest credible fix. Do not combine unrelated findings merely to reduce the comment count.

When submission is explicitly authorized:

1. Refetch the PR head and diff immediately before posting. If the head changed, discard stale anchors and re-review the affected diff.
2. Use the provider reference to submit one review containing all inline findings, so they share one disposition and do not produce a burst of separate notifications.
3. Use a comment disposition unless the user explicitly requested approval or changes. Do not infer a disposition from finding severity.
4. Put only the concise summary and unanchorable findings in the overall review body. Explain why each unanchorable finding could not be placed inline.
5. Verify the returned review and every inline comment by stable ID or URL. After an ambiguous timeout or transport failure, inspect the PR before retrying so a successful write is not duplicated.

## Report the Findings

Lead with fresh review findings ordered by severity:

| Severity | Path:Line | Placement | Status / URL |
|---|---|---|---|

Use `inline`, `overall`, or `local-only` for Placement. For an overall finding, state why no changed-line anchor applies.
