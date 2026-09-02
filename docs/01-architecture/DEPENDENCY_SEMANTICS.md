# Dependency Semantics

## Default

A normal task uses `ALL_SUCCEEDED`: every required upstream TaskRun must succeed.

## Proposed built-in trigger rules

Keep the initial rule set intentionally small and explainable:

- `ALL_SUCCEEDED`
- `ALL_TERMINAL`
- `ANY_SUCCEEDED`
- `ANY_FAILED`
- `NONE_FAILED`

Custom arbitrary expressions are deferred until there is a proven need.

## Skip propagation

`SKIPPED` is not success or failure. The downstream trigger rule decides whether skipped upstreams satisfy the predicate.

Every skip must contain a reason, for example:

- branch not selected,
- dependency predicate impossible,
- workflow cancellation,
- fail-fast prevented scheduling.

## Branches

A condition/gate records its chosen branch as a durable decision. Untaken branch nodes are marked skipped. On engine recovery, Andormu reads the recorded branch decision; it does not recompute an unrecorded external condition.

## Joins

Joins are ordinary dependency evaluation. This avoids a separate hidden “join scheduler”.

## Finalizers

Finalizers use dedicated semantics described in `FINALIZERS.md`; they should not be modeled as a fragile combination of ordinary trigger rules.
