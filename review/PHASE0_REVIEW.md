# Phase-0 Design Review

## Current status

**Not ready to code.** This baseline is designed to support review.

## Critical review questions

### State and dependency

- [ ] Is the WorkflowRun state model minimal but sufficient?
- [ ] Are TaskRun and TaskAttempt states separated correctly?
- [ ] Is `SKIPPED` semantics precise enough?
- [ ] Is the proposed trigger-rule set sufficient for Phase 1?

### Retry and timeout

- [ ] Do retry counters reset on redrive?
- [ ] Which timeout clocks pause during suspension?
- [ ] Is heartbeat timeout required in Phase 1?
- [ ] Who classifies backend failures: adapter, Compute, Andormu, or a layered combination?

### Cancellation/finalization

- [ ] Does suspend allow current attempts to finish?
- [ ] Exactly which finalizers run on cancel, timeout, fail-fast, and terminate?
- [ ] Can finalizer failure change the primary workflow outcome?

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
- [ ] How is admission waiting represented without leaking scheduler semantics?


### Service / asynchronous task execution

- [ ] Is `Task = logical unit of work` accepted as the canonical abstraction?
- [ ] Which async completion mechanisms are mandatory in Phase 1: callback, event, poll, watch?
- [ ] Where does capability -> execution target resolution live?
- [ ] What is the minimum execution-adapter protocol for persistent services?
- [ ] How is an unobservable/lost async operation represented?

### Backpressure / workflow granularity

- [ ] Are per-capability/service concurrency limits core Andormu policy?
- [ ] Is cross-tenant logical fairness owned by Andormu or a shared admission service?
- [ ] Is `DISPATCHABLE` a persisted state or an internal condition?
- [ ] What workflow granularity guidance/conformance tests prevent millions of tiny inefficient WorkflowRuns?
- [ ] What bounded fan-out safeguards are required for file/data-loop use cases?

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
