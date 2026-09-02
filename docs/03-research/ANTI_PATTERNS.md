# Anti-patterns to Avoid

## AP1 — Kubernetes object leakage

If `WorkflowSpec` requires Pod affinity/tolerations/Job fields, Andormu stops being a portable shared engine.

## AP2 — One state enum for every reason

Do not create dozens of states such as `WAITING_FOR_GPU`, `IMAGE_PULL_ERROR`, `NETWORK_ERROR`. Use simple lifecycle states plus structured reasons/failures.

## AP3 — Retry overwrites history

A retried task must retain every attempt.

## AP4 — Exactly-once marketing

Never claim exactly-once arbitrary side effects. Use at-least-once dispatch plus idempotency/compensation.

## AP5 — “Retry workflow” without defining reuse semantics

Users must know whether successful tasks run again, whether definition version changes, and whether retry counters reset.

## AP6 — Hidden finalizers

Cleanup must not depend on a clever trigger-rule combination that users cannot reason about.

## AP7 — Arbitrary runtime DAG mutation

Unrecorded graph changes make crash recovery non-reproducible.

## AP8 — Resource manager creep

Logical `max_parallelism` is Andormu; GPU placement, quota, preemption, and queue policy are Compute Platform.

## AP9 — Large payloads in workflow history

Store references/digests for large artifacts.

## AP10 — Coding before semantics

State/recovery bugs become architectural debt quickly. Do not let an implementation prototype silently define the contract.
