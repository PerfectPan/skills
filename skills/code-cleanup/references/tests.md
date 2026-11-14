# Reviewing test value

Judge assertions and exercised code, not file length, test names or whether a test looks static.

A test title is not a requirement authority. Trace a claimed obligation to supported behavior, documentation or a
credible regression. When all existing assertions already have stronger proof, an unrelated coverage gap does not
make those assertions nonredundant. Record the gap separately; adding a copied inventory or new assertions to justify
keeping the old test is new test work, not preservation evidence for the cleanup.

## Authoring and retention

A useful test protects an observable behavior, invariant or independent compatibility obligation and fails under a
credible regression. Before adding or retaining another proof of the same behavior, identify what the current tests
miss. A lower-level test may protect a domain edge that a happy-path integration test never exercises; a higher-level
test may protect wiring, transport or resource lifetime that a unit test cannot see.

Keep distinct security, authorization, protocol, public API, configuration/default, migration, persistence,
architecture, packaging and release obligations. Identity, exact bytes, source structure or invocation order can be
the actual requirement. A structural assertion is justified when it guards an independent rule and tolerates
irrelevant identifier-only refactors. Slowness is a cost to investigate, not sufficient grounds for deletion.

## Candidate patterns

Investigate assertions that compare a value with itself or derive the expected answer from the subject, duplicated
inventories without an independent requirement, mocks that implement the behavior being asserted, and checks of
private call shape or export existence with no production obligation. A round trip can be useful when the two sides
have independent implementations or an external format requirement; it is not automatically self-comparison.

Inspect fixtures and negative cases closely. The fixture must not supply the ordering, persistence receipt or state
transition that the owner is supposed to produce. A rejection must reach the guard named by the test. An assertion
that fails on another field or never exercises its named branch needs repair, not automatic deletion.

## Decide per assertion

- **Retain:** name the unique regression or boundary obligation.
- **Repair:** retain the obligation, but correct a vacuous assertion, unreachable input or misleading fixture.
- **Consolidate:** move a unique edge into an existing owner test or parameter table, then retire redundant setup.
- **Delete:** show which existing assertion exercises the same obligation, or demonstrate that no supported
  production/compatibility obligation exists. Check history before treating an old test as obsolete.

A replacement test must actually traverse the relevant owner without mocking the behavior away. Compare inputs,
failure modes and assertions individually; broad integration coverage or a green percentage is not a mapping.
Preserve distinctions such as missing versus malformed data, restart versus in-process behavior, or rejected writes
versus successful writes. Keep one useful failure location instead of trading many cheap focused tests for an opaque,
expensive scenario merely to reduce counts.

Before sharing fixtures, compare key insertion order, omitted fields, object identity, mutation and lifetime. Preserve
deliberate variants and suite-specific failure injection; use fresh data where isolation requires it. Replacing every
variant with one constant can leave tests green while removing the behavior they used to exercise.

## Cleanup evidence

Record original assertions and their surviving homes before deletion. Check test discovery and CI routing, then run
the owner and affected boundary suites. If test removal makes a fixture/export unused, inspect production consumers
and public surface before deleting it. Never add an export, flag or wrapper only to preserve an implementation-shaped
test; test through the real owner when that keeps the same obligation.

Report a moved assertion as retained coverage, not as junk removed. State baseline failures and unavailable proof
separately. If only a sample was reviewed, list the sampled owners rather than claiming the whole suite is excessive.
