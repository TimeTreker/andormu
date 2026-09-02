# Finalizers and Cleanup

Cleanup must be first-class.

## Workflow finalizers

A WorkflowSpec may declare finalizer nodes that are evaluated after normal execution reaches a completion/cancellation boundary.

Typical uses:

- release temporary resources,
- publish final status,
- close external leases,
- cleanup scratch storage.

## Desired semantics

- Finalizers are not normal success-path dependencies.
- Graceful cancellation still runs eligible finalizers.
- Hard termination does not guarantee them.
- Finalizer failure is visible separately from the primary workflow failure.
- The workflow outcome policy must state whether finalizer failure changes the final WorkflowRun outcome.

## Scope

A later design may add setup/finalizer scopes similar to Airflow setup/teardown. Phase 1 should favor workflow-level finalizers unless task-scoped cleanup has a concrete requirement.
