# ADR-0017: Andormu owns logical admission; Compute owns physical admission

**Status:** Accepted

## Context

Dependency-ready work can overwhelm a constrained downstream service or worker pool even when physical compute resources are healthy.

Conversely, a logically admitted GPU task may still need to wait for quota, placement, topology, or accelerator availability.

These are different scheduling problems.

## Decision

Andormu owns **logical dispatch admission** for workflow/task progression and downstream protection.

Compute/cloud platforms own **physical resource admission and placement**.

Logical admission may include workflow/capability concurrency, dispatch pacing/rate limits, queue-depth/backpressure policy, bounded fan-out, and priority/fairness policy where separately accepted.

Physical admission includes hardware quota, queues, CPU/GPU/NPU availability, Kubernetes/Slurm/Ray placement, topology, reservations, preemption, and GPU virtualization decisions.

## Consequences

- Dependency-ready does not imply dispatch-now.
- The exact persisted representation of admission waiting (`READY`, `DISPATCHABLE`, reason/condition, etc.) remains a Phase-0 state-model decision.
- Andormu must not evolve into a GPU/cluster scheduler.
- Compute admission state must be normalized without leaking backend-native scheduling semantics into the canonical DAG.
