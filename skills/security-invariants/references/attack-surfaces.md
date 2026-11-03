# Attack Surfaces

Read the sections that match the system under review. These are questions to falsify, not a checklist
whose completion proves security.

## Fatal Input and Parser Boundaries

- Can request, database, federation, or configuration data reach `panic`, `unwrap`, `expect`, an
  assertion, unchecked indexing, process exit, or an exception that terminates the worker?
- Does locally accepted input enter a stricter dependency parser or serializer later?
- Does the production failure mode abort the process, kill one task, retry forever, or corrupt state?
- Are background jobs and notification paths reachable by an unprivileged actor?

## Structured Values and Canonicalization

- Are scopes, roles, groups, domains, origins, hosts, paths, and identifiers parsed into components
  before comparison?
- Is exact membership accidentally implemented with substring, prefix, suffix, replacement, or a
  delimiter-sensitive string operation?
- Does each comparison use the protocol's defined equality? Normalize case, Unicode, trailing dots,
  default ports, percent encoding, and duplicates only where the protocol permits it; preserve opaque
  identifiers byte-for-byte and reject ambiguous encodings or separators.

## Authority and Lifetime

- Is output authority the intersection of the subject, actor, client, requested target, resource,
  account policy, and server policy?
- Can exchange, refresh, delegation, or impersonation copy permissions the receiving party lacks?
- Can a derived artifact outlive the account, session, delegation, consent, client limit, absolute
  renewal limit, or other authority that permits it? Distinguish the short redemption window of a
  one-time credential such as an authorization code from the lifetime of the authority it conveys.
- Are optional claims and custom attributes released only for exact authorized scopes?

## Artifact Purpose and Replay

- Are cookies, tokens, states, magic links, and encrypted blobs bound to a purpose, issuer, audience,
  client, redirect target, and version where applicable?
- Can a valid artifact be renamed, moved to another header or cookie, or decoded by another consumer?
- Does encryption authenticate context as well as plaintext, or do all consumers accept the same
  plaintext shape under shared keys?
- Can an application, relying party, or intermediary observe a credential that authenticates to a
  more privileged service?

## Identity Namespaces

- Are local user IDs, provider subjects, emails, usernames, session IDs, and client IDs distinct
  types and lookup paths?
- Is a provider identifier always qualified by provider or issuer?
- Can collision, linking, migration, or fallback logic cross namespaces?

## Proxy and Deployment Trust

- Which immediate peers are trusted, and which headers can each peer set?
- Does the application parse the whole forwarded chain or merely trust the leftmost value?
- Do documented proxies overwrite hostile incoming headers, and is that behavior verified rather
  than assumed?
- Can direct network access bypass the proxy, or can multiple trusted hops disagree on syntax?
- Are scheme, host, original URI, and client IP used in authorization or redirects?

## Stateful Defenses

- What keys counters, locks, replay caches, and revocations: IP, account, credential pair, session,
  client, or a combination?
- Can success by one identity reset failures caused against other identities?
- Do concurrent requests race between check, claim, update, and deletion?
- Can retries or reordering revive consumed state?
- Does one defense hide a weakness in another, and can both be bypassed by the same input?

## Composition

After testing findings separately, combine their prerequisites. A validation bypass may expose a
fatal parser; spoofed identity metadata may defeat a counter; a low-privilege application may turn a
replayable artifact into a privileged session. Report the chain without inflating the independent
severity of each link.
