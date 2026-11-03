# Evidence and Severity

Security reports must make it easy to distinguish what happened from what the reviewer predicts.

## Evidence Classes

- **Live reproduction**: observed through the real process or protocol boundary in an isolated
  environment.
- **Integration reproduction**: observed across the relevant production components with controlled
  infrastructure.
- **Unit or property proof**: executed against the real functions or dependency behavior, but not the
  complete deployment path.
- **Code-only trace**: reachable path established by inspection; runtime behavior or prerequisites
  remain unverified.
- **Candidate**: suspicious pattern without a complete path. Do not report it as a vulnerability.

For a split proof, name every proven and unproven link. A fatal parser plus a separately observed
process-abort policy is strong evidence, but it is not an end-to-end reproduction.

## Finding Record

Before reporting, establish:

- violated invariant;
- attacker and required privileges;
- relevant configuration and deployment assumptions;
- controlled input and complete path to the security decision;
- observed behavior and evidence class;
- confidentiality, integrity, or availability consequence;
- affected scope and plausible blast radius;
- smallest credible repair and regression test.

Do not use a configuration prerequisite to dismiss a finding when that configuration is supported or
documented. Do not describe a supported feature as universally vulnerable when exploitability depends
on an intermediary behaving unsafely.

## Severity and Confidence

Rate severity from consequence, exploitability, privileges, user interaction, affected scope, and
recoverability. Rate confidence separately from evidence quality. A high-impact code-only path can be
high severity with moderate confidence; a fully reproduced information leak can be medium severity
with high confidence.

Avoid precise CVSS scores until affected versions and deployment assumptions are verified. Never
raise severity merely because several findings were discovered together.

## Reporting Discipline

- Keep independent root causes separate; group variants that share one enforcement failure.
- State safe configurations and existing compensating controls without treating them as fixes.
- Call out contradictions and gaps in the reproduction notes.
- Keep credentials, personal data, and undisclosed exploit details in the repository's private
  security channel.
- Do not claim a release is affected when only a development revision was tested.
