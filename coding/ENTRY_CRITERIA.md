# Phase 1 Coding Entry Criteria

All critical items must be accepted before implementation begins.

## Goal Gate

- [ ] Product goal and non-goals accepted.
- [ ] Domain/platform boundary accepted.

## Semantic Gate

- [ ] WorkflowSpec / TaskSpec conceptual model accepted.
- [ ] WorkflowRun / TaskRun / TaskAttempt states accepted.
- [ ] Dependency/skip semantics accepted.
- [ ] Retry/backoff/timeout semantics accepted.
- [ ] Failure propagation accepted.
- [ ] Suspend/cancel/terminate semantics accepted.
- [ ] Finalizer semantics accepted.
- [ ] Redrive/restart semantics accepted.
- [ ] Dynamic graph semantics accepted.
- [ ] At-least-once/idempotency semantics accepted.

## Architecture Gate

- [ ] Persistence + durable event history direction accepted.
- [ ] Reconciliation model accepted.
- [ ] Execution adapter boundary accepted.
- [ ] Compute/resource boundary accepted.
- [ ] HA/scaling assumptions reviewed.

## Product Gate

- [ ] Run List/Run Detail UX accepted.
- [ ] Recovery operation UX accepted.
- [ ] RBAC/audit requirements accepted.

## Contract Gate

- [ ] Contract versioning policy accepted.
- [ ] Bronze Dragonflight shared-contract ownership agreed.
- [ ] Critical open questions resolved or intentionally deferred.

Only then create implementation directories and schemas.
