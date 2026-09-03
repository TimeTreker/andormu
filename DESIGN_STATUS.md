# Design Status

**Phase:** 0 — Goal / Architecture / Product / Contract design

**Coding:** blocked by design gate

## Accepted boundary decisions

- Timeways is the vendor-neutral execution platform of Bronze Dragonflight; Andormu is its durable control plane.
- Bronze Dragonflight owns the canonical workflow/task execution contract rather than adopting a cloud/vendor workflow model as canonical.
- Andormu must preserve semantic portability across supported cloud and on-prem execution environments.
- Domain semantics and domain workflow selection stay in domain platforms.
- Andormu owns logical dispatch admission/backpressure; physical compute/resource admission and placement stay in Compute Platform.
- Core contracts are backend-neutral, not Kubernetes-, Slurm-, Ray-, or cloud-vendor-native.
- A Task is logical work, not a process; persistent services, deferred/asynchronous operations, workers, data engines, and compute jobs are execution realizations of TaskAttempts.
- DAG node semantics describe logical work/control flow, not deployment/runtime process types.
- TaskDefinition is the reusable logical contract; TaskSpec is one exact occurrence in a WorkflowSpec.
- ExecutionBinding is the immutable environment mapping from exact Capability to provider, completion model, adapter, and stable ExecutionTarget; bindings are resolved outside the DAG and pinned per run.
- Provider class (`SERVICE`, `WORKER`, `COMPUTE`, `EXTERNAL_RUNTIME`) and completion model (`INLINE`, `DEFERRED`) are independent dimensions.
- The bag-processing-v3 and normal-file-processing-v5 workflows are canonical regression scenarios for later execution/state design.
- Workflow-submission idempotency and TaskAttempt dispatch idempotency are distinct domains.
- Execution-contract validation is separate from a business result-validation DAG task.
- Standard infrastructure such as Kafka/MQ, Flink/Spark, Kubernetes, GPU virtualization, object storage, databases, and observability systems is reused rather than reimplemented by Andormu.
- External workflow/data engines may be used as opaque task executors; Andormu does not duplicate ownership of the same internal DAG.
- Phase 0 is design-only.

See ADR-0013 through ADR-0018 for the accepted execution-platform boundary.

## Major proposed decisions still requiring review

- Immutable ExecutionSnapshot details.
- TaskRun/TaskAttempt state model and exact attempt-creation boundary.
- Reconciliation + durable event journal architecture.
- At-least-once dispatch protocol details.
- Redrive behavior.
- Dynamic expansion rules.
- Finalizer rules.
- Detailed state/trigger-rule sets.
- ExecutionBinding resolver implementation and compatibility negotiation details.
- Logical admission mechanics, capacity signals, and cross-tenant fairness scope.
- Async/deferred completion callback/poll/watch protocol details.
- Minimum adapter mappings for service, data-engine, and compute backends.

See `review/PHASE0_REVIEW.md`.
