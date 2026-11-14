# Consolidating repeated code

Similar syntax is a search signal. Shared behavior and a legitimate shared owner justify consolidation.

## Compare meaning

Read the detector configuration: included roots, exclusions, matching thresholds and invocation. Report those limits;
unscanned code and below-threshold repetition remain possible even when a configured gate passes.

Read the complete implementations, production callers and existing tests. Compare accepted inputs, outputs,
normalization, mutation/object identity, errors and their identity/order, defaults, async cancellation, resource
lifetime and persistence boundaries. Record intentional differences. Zero token clones does not prove zero repeated
behavior; a clone report does not prove two implementations have the same obligations.

Compare when inputs and errors are captured, not just the final expression. Reading mutable state at construction,
subscription or event time can produce different results. Distinguish byte-identical output from equivalent execution:
two commands can run the same way while changing displayed text, stored data or a compatibility requirement.

Ask why each implementation changes. Two real consumers of the same policy can share it. Similar short adapters
around different ports or contexts may be clearer separately, especially when their lifetimes or compatibility
requirements differ. Consolidation must reduce maintenance work after accounting for extra indirection, parameters
and coupling.

## Choose the owner

Prefer an existing suitable implementation or dependency already used by the project. Verify its actual semantics
before substitution; a library utility's edge cases may differ from the current helper. Do not add a dependency just
to remove a few lines without a concrete benefit and the repository's required dependency review.

If extraction is justified, place it at the narrowest existing boundary that all real consumers may import. Keep
business rules with their domain owner and platform mechanics free of business types. Follow public module boundaries;
similar code is not permission for deep imports, new dependency cycles, broader exports, or a catch-all utility module.

Trace imports in both directions before choosing one existing module as the shared owner. If that introduces a cycle
or expands a published API, find a narrower permitted owner or retain the repetition until that change is justified.

Prefer moving the implementation and updating callers directly. Avoid a compatibility forwarding layer for a private
path unless there is a real supported caller. Leave independent domain concepts separate when sharing would require
inappropriate ownership changes or a generic framework larger than the repetition.

## Preserve behavior and proof

Write the equivalence conditions in the candidate ledger before editing. If implementations differ intentionally,
keep them separate or share only a smaller common operation whose inputs and outputs can express both cases simply.
Do not silently choose one behavior as the new default. A desired behavior change needs its own explicit scope and
the repository's design process.

Move all necessary edge assertions to the surviving owner, retaining callers' independent wiring and lifecycle
tests. Verify public declarations and artifact entrypoints where affected, and check exact-path configuration,
dynamic imports and fixtures as well as compiler-resolved imports. A reduced source count with an expanded public API
or a less accurate test boundary is not automatically an improvement.

Report consolidations with before/after owners, real consumers and preservation evidence. Also report plausible
duplicates deliberately retained and the semantic or ownership difference that prevents merging them.
