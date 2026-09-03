# Phase-0 Design Review

## Current status

**Not ready to code.** This baseline is designed to support review.

## Accepted platform anchors

The following are no longer open scope questions:

- [x] Timeways is the Bronze Dragonflight vendor-neutral execution platform; Andormu is its durable control plane.
- [x] Bronze Dragonflight owns the canonical workflow/task execution contract.
- [x] Cross-cloud/on-prem semantic portability is a platform requirement.
- [x] Domain platforms own business workflow selection/compilation.
- [x] `Task = logical unit of work`, not process/container/Pod.
- [x] DAG task semantics are separate from service/worker/data-engine/compute execution realization.
- [x] Andormu owns logical dispatch admission/backpressure; Compute owns physical resource admission/placement.
- [x] Standard infrastructure is reused rather than reimplemented by Andormu.
- [x] External engines normally remain the authority for their own internal DAGs and are integrated as opaque task executions.

These anchors are recorded in ADR-0013 through ADR-0018.

## Critical review questions

### Production reference workflow

Use `docs/01-architecture/PRODUCTION_DATA_LOOP_EXAMPLE.md` as a recurring conformance scenario.

- [ ] Can one vendor-neutral `WorkflowSpec` model the bag and normal-data workflows without cloud/Kubernetes/Flink-specific graph semantics?
- [ ] Can business-required subsets of safety/preprocess/decode/encode/index/result-check nodes be expressed cleanly without a universal mega-workflow?
- [ ] Is workflow-submission idempotency sufficient for duplicate OSS/Kafka trigger delivery?
- [ ] Is `1 artifact = 1 WorkflowRun` acceptable for operationally meaningful large files, with batching/bounded fan-out guidance for tiny files?

### State and dependency

- [ ] Is the WorkflowRun state model minimal but sufficient?
- [ ] Are TaskRun and TaskAttempt states separated correctly?
- [ ] At what exact point after logical admission is a TaskAttempt created?
- [ ] Is `SKIPPED` semantics precise enough?
- [ ] Is the proposed trigger-rule set sufficient for Phase 1?

### Retry and timeout

- [ ] Do retry counters reset on redrive?
- [ ] Which timeout clocks pause during suspension?
- [ ] Is heartbeat timeout required in Phase 1?
- [ ] Who classifies backend failures: adapter, Compute, Andormu, or a layered combination?
- [ ] What retry behavior is safe after an unobservable/unknown-outcome execution?

### Cancellation/finalization

- [ ] Does suspend allow current attempts to finish?
- [ ] Exactly which finalizers run on cancel, timeout, fail-fast, and terminate?
- [ ] Can finalizer failure change the primary workflow outcome?
- [ ] How are logical cancellation intent and still-running backend execution represented separately?

### Dynamic graph

- [ ] Is dynamic fan-out needed in Phase 1 or Phase 2?
- [ ] What is the maximum/guardrail for graph expansion?
- [ ] How are nested subworkflow cancel/failure states propagated?

### Persistence/event model

- [ ] Is the append-only journal the source of truth, or an audit journal beside transactional state?
- [ ] What aggregate ordering guarantees are required?
- [ ] How are durable timers represented?

### Resource/execution boundary

- [ ] Is ResourceIntent owned by Bronze Dragonflight shared contracts or Compute Platform contracts?
- [ ] Does an execution adapter contact Compute directly, or through a standard Execution Gateway?
- [ ] How is physical-admission waiting represented without leaking scheduler-native semantics into TaskRun?
- [ ] Which backend-native details are retained only as opaque references/log links versus normalized observations?

### Task execution protocol

- [ ] What is the minimum common adapter protocol: dispatch, observe, cancel, optional renew/heartbeat?
- [ ] Which deferred-completion mechanisms are mandatory in Phase 1: callback, event, poll, watch?
- [ ] Is inline vs deferred completion modeled as execution-contract capability rather than DAG node kind?
- [ ] Where does capability -> execution target resolution live?
- [ ] What is the minimum execution-adapter protocol for persistent services?
- [ ] How are Flink/Spark/data-engine jobs mapped without duplicating their internal DAG?
- [ ] How is an unobservable/lost async operation represented?
- [ ] What is the output-contract validation boundary before a TaskRun can release downstream dependencies?

### Logical admission / workflow granularity

- [ ] What minimum per-workflow and per-capability concurrency policies are required in Phase 1?
- [ ] Which capacity signals are required or optional: static limit, worker slots, service hints/429, adapter feedback, queue depth?
- [ ] Is cross-tenant logical fairness implemented in Andormu Phase 1 or delegated to a shared admission component?
- [ ] Is `DISPATCHABLE` a persisted state or an internal/durable admission condition?
- [ ] What workflow granularity guidance/conformance tests prevent millions of tiny inefficient WorkflowRuns?
- [ ] What bounded fan-out safeguards are required for file/data-loop use cases?

### Standard infrastructure dependency policy

- [ ] Which infrastructure capabilities are mandatory production dependencies versus optional adapters?
- [ ] What storage guarantees are required from the durable state store (ACID, uniqueness, conditional update, etc.) without binding canonical semantics to one database product?
- [ ] What event-transport guarantees are required without making Kafka offsets/messages orchestration truth?
- [ ] What conformance tests ensure a cloud/on-prem adapter preserves Bronze TaskAttempt semantics?

### Data/security

- [ ] Inline result size limit?
- [ ] SecretRef standard?
- [ ] ArtifactRef standard?

### Product

- [ ] Are Redrive and Restart enough for Phase 1?
- [ ] Is targeted rerun intentionally deferred?
- [ ] Is visual authoring intentionally deferred?

## Design Gate

Phase 1 coding begins only when the critical questions above are either accepted or explicitly deferred with ADRs.
