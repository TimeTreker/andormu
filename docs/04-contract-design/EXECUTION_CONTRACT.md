# Task Execution Adapter Contract

This document records protocol semantics only. Exact messages, enums, and transports remain unfrozen.

## Boundary

Andormu dispatches a logical `TaskAttempt` through the adapter selected by the pinned `ExecutionBinding`. The adapter translates a small vendor-neutral protocol into service, worker, compute, or external-runtime mechanics.

Backend-native phases and identifiers remain diagnostic observations; they do not become canonical DAG states.

## Pinned binding context

Before dispatch, the ExecutionSnapshot has fixed:

- exact TaskDefinition and Capability;
- exact ExecutionBinding revision;
- provider class and completion model;
- adapter and stable ExecutionTarget;
- supported observation/cancellation behavior;
- applicable AdmissionPolicy revision;
- optional ResourceIntent for the physical execution provider.

## Commands

- `Dispatch` one TaskAttempt;
- `Observe` a deferred TaskAttempt;
- `Cancel` a TaskAttempt;
- optional heartbeat/renew only where the binding's protocol requires it.

Callback, event, poll, and watch are observation mechanisms for `DEFERRED` completion, not separate task kinds.

## Dispatch request semantics

A request carries:

- stable WorkflowRun, TaskRun, and TaskAttempt identities;
- exact TaskDefinition/Capability and pinned ExecutionBinding references;
- compact resolved inputs and ArtifactRefs;
- logical timeout, retry, cancellation, and deadline context needed by the adapter;
- trace/correlation context;
- secret references;
- stable dispatch idempotency identity;
- optional ResourceIntent supplied by the binding for physical admission.

The request does not carry the entire WorkflowSpec or allow the adapter to reinterpret DAG dependencies.

## Dispatch response by completion model

### INLINE

The dispatch interaction returns either:

- a terminal success result with declared outputs; or
- a structured terminal failure/rejection.

### DEFERRED

The dispatch interaction returns:

- accepted or already-existing status;
- opaque ExecutionHandle;
- current generic non-terminal observation;
- backend/log references where available;
- structured rejection if work was not accepted.

Terminal completion is learned later through one or more observation mechanisms declared by the binding.

## Observation semantics

Every observation must correlate to the exact TaskAttempt and be safe under duplication and reordering. It may contain:

- normalized generic phase/outcome;
- declared outputs or ArtifactRefs on success;
- structured failure on terminal failure;
- opaque backend references and log links;
- monotonic backend observation/version information where available.

The later unified execution-protocol review must define precedence for callback/poll/watch races and stale observations.

## Contract validation before logical success

An adapter reporting success is necessary but not sufficient. Andormu must:

1. correlate the observation to the exact attempt;
2. verify terminal outcome shape;
3. validate required outputs against the pinned TaskDefinition contract;
4. persist the accepted logical outcome before releasing downstream dependencies.

Missing or invalid required outputs produce an execution-contract failure. This is separate from a downstream business validation task such as `result_check`.

## Idempotency invariant

Repeated dispatch for one TaskAttempt identity must return/observe the same external execution and must not create another logical attempt. An execution retry receives a new TaskAttempt identity.

This invariant is distinct from deduplicating repeated WorkflowRun submissions at the workflow API boundary.

## Cancellation

Cancellation support and guarantees are declared by the binding. `BEST_EFFORT` may be valid for an external service or compute backend, but Andormu records cancellation intent durably before issuing the adapter command and continues reconciliation until later state semantics define the outcome.

## Explicitly deferred decisions

The following belong to the next unified execution-protocol and TaskAttempt reviews:

- exact messages and transport;
- generic non-terminal and terminal phase enums;
- callback authentication and replay protection;
- observe cadence, leases, heartbeat, and lost-handle rules;
- cancel race outcomes;
- timeout ownership and late completion;
- normalized failure taxonomy;
- state-transition persistence semantics.
