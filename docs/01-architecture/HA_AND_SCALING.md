# High Availability and Scaling Design Questions

Phase 0 defines semantic requirements without choosing an implementation stack.

## Required properties

- horizontally scalable reconciliation,
- no single in-memory owner of workflow truth,
- safe duplicate reconciliation,
- bounded work per reconcile cycle,
- durable timers/retry scheduling,
- leader/lease or partitioning strategy that prevents conflicting side effects,
- backpressure when adapters or storage are unhealthy.

## Candidate execution architecture

A common scalable shape is:

```text
API / ingest
    ↓
Durable state + event journal
    ↓
Run work queue / partitioning
    ↓
Stateless reconcilers
    ↓
Idempotent dispatch adapters
```

Flyte Propeller's work-queue/reconciler design is a useful reference, but Andormu should not commit to Kubernetes CRDs as the persistence mechanism.

## Open decisions

- database-backed lease vs distributed queue ownership,
- event-sourced reconstruction vs transactional materialized state + audit journal,
- timer wheel/service design,
- partition key (namespace, run id, hash),
- cross-region / multi-cluster control plane in later phases.
