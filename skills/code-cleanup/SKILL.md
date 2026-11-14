---
name: code-cleanup
description: Audit low-value tests and duplicated implementations, then apply evidence-backed cleanup when requested. Use for test-pruning, test-value reviews, or finding and consolidating repeated code; ordinary feature work and filename-only edits do not require this workflow.
---

# Code Cleanup

Reduce maintenance work while preserving observable behavior and independent regression protection. A smaller diff,
test count or coverage percentage is not evidence that a cleanup is correct.

## Select the scope

Read the repository instructions, current change state, architecture and relevant test/build commands. Record the
base revision and the owners being examined. Respect read-only requests; when implementation is already authorized,
the skill does not introduce another approval requirement. Follow the repository's design and delivery gates.

For an audit-only request, deliver the decision ledger, examined and unexamined scope, retained apparent duplicates,
and checks actually run versus proposed. Stop before implementation; a recommendation is not verified delivery.

- For test value or pruning, read [test review](references/tests.md).
- For repeated implementations, read [code consolidation](references/consolidation.md).
- When both are requested, use both: deleting a redundant implementation can justify moving its tests to the
  remaining owner. It does not make those original assertions worthless.

## Establish evidence before editing

Inventory the selected owners, their production callers, tests and test support. Search and clone detectors produce
candidates, not decisions. Read each candidate in full, its implementation, meaningful callers and overlapping proof;
use history to understand compatibility or regression obligations that are not obvious from today's code.

Keep a compact decision ledger. For every proposed change, record:

| Item | Required evidence |
| --- | --- |
| Location and obligation | Exact function or assertion, its owner and externally meaningful behavior |
| Failure or equivalence | For a test, the regression it detects; for code, equivalent behavior and intentional differences |
| Callers and overlap | Production consumers and the other implementation or test being compared |
| Decision | Retain, repair, consolidate or delete, with the reason |
| Preservation | Where each necessary behavior and assertion will remain; for dead code, evidence that no supported obligation remains |
| Verification | Focused owner/sibling checks and any compatibility checks affected |

Missing evidence means retain or investigate further. Retain is a useful result. Report what was examined and what
was not; sample findings do not establish a complete repository audit. No deletion quota or required abstraction count.

## Apply an evidenced change

Work in coherent owner batches. Record relevant baseline results before edits; existing failures are not candidates
for removal merely because they fail. Preserve public APIs, stored formats, error semantics, ordering and lifetime.
Keep checks and coverage thresholds unchanged.

Move any unique proof before deleting its former owner. Remove obsolete private support or test-only production
seams only after checking all callers and public compatibility. A test disappearing from a glob is lost coverage,
not a passing test. Stop concurrent test runs before mutating their checkout.

Check cited paths and proposed imports against the real tree and dependency graph. Derive validation commands from
existing tests and the repository's runner; distinguish a proposed new test from one that already exists.

Run the focused owner and caller tests, then the repository's required final checks. For changed bug behavior, obtain
the repository's required failing-before/passing-after evidence. For uncertain retained proof, use a small deliberate
negative control in an isolated copy: it must fail for the intended behavior, not setup or a different guard. Restore
the copy and record the result. Refactors need preservation evidence, not ceremonial tests for each moved file.

Review the final diff for behavior lost with deleted code or assertions, orphaned support and unrelated changes.
Report retained obligations, removals versus moved proof, verification results and limitations. Separate production
and test/support line changes, and distinguish measured runtime from estimates. Commit, push and merge only within
the user's authorization and the repository workflow.

## Provenance

The test-review approach is informed by OpenClaw's
[test-audit](https://github.com/openclaw/openclaw/blob/main/.agents/skills/test-audit/SKILL.md) and its
[evidence checklist](https://github.com/openclaw/openclaw/blob/main/.agents/skills/test-audit/references/audit.md).
This skill uses original instructions and the target repository's tools, ownership rules and delivery gates.
