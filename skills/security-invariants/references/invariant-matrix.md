# Invariant Matrix

Use a matrix whenever several paths should enforce the same security rule. The rows are entry points
or artifact transitions. The columns are required invariants. Derive both from the system instead of
copying a fixed OAuth checklist.

## Build the Matrix

1. Find every public entry point and every internal producer or consumer of the protected artifact.
2. Include alternate protocols, legacy routes, refresh and exchange paths, administrative paths,
   background jobs, federation callbacks, and optional configurations.
3. Define each column as a falsifiable rule with one enforcement owner.
4. Fill a cell only from code evidence or a test. Use `?` for uninspected and `N/A` only with a reason.
5. Investigate inconsistent cells first. A check in one path is evidence that its absence elsewhere
   matters.

Example shape:

| Path | Principal valid | Client valid | Policy | Authority narrowed | Audience bound | Lifetime bounded |
| --- | --- | --- | --- | --- | --- | --- |
| Primary issuance | pass | pass | pass | pass | pass | pass |
| Alternate grant | pass | pass | ? | pass | pass | pass |
| Refresh | pass | pass | pass | pass | pass | ? |
| Exchange | pass | pass | ? | ? | pass | ? |

The matrix is a reasoning aid, not a compliance score. A row can pass every named column while the
column set is incomplete.

## Turn It Into Tests

For each invariant, test at least one allowed and one denied case through every applicable row. Add
cross-product cases only where interactions change the outcome; do not create a slow exhaustive suite
without a risk model.

Useful test forms include:

- table-driven tests sharing the same policy cases across handlers;
- property tests proving output authority is a subset of every input authority;
- sequence tests covering failure, success, reset, retry, replay, expiry, and revocation;
- integration tests proving deployment metadata such as proxy headers reaches the same decision;
- differential tests asserting equivalent paths return equivalent security decisions.

A new route is incomplete until it appears in the relevant matrix and executes the shared cases.
