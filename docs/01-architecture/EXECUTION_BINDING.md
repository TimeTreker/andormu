# ExecutionBinding Design

This document defines Phase-0 semantic intent only. It does **not** freeze a storage or wire schema.

## Purpose

`ExecutionBinding` is the explicit boundary between logical task meaning and environment-specific execution realization.

```text
TaskDefinition -> Capability -> ExecutionBinding -> Execution Adapter
```

It answers:

> In this environment, how is this exact capability executed and observed?

## Conceptual contents

An immutable binding revision records:

- binding identity, revision, and environment scope;
- exact capability requirement;
- provider class;
- completion model;
- adapter reference;
- stable logical ExecutionTarget;
- observation methods supported for deferred completion;
- cancellation support/guarantee;
- optional AdmissionPolicy reference/key;
- optional ResourceIntent passed to a compute provider;
- bounded operational metadata such as log/reference capabilities.

This is a semantic checklist, not a proposed serialization.

## Two independent execution dimensions

### Provider class

Phase 0 uses a small vendor-neutral classification:

```text
SERVICE
WORKER
COMPUTE
EXTERNAL_RUNTIME
```

Provider class describes the kind of execution-plane realization. It does not create four kinds of business DAG node.

### Completion model

Phase 0 uses only:

```text
INLINE
DEFERRED
```

- `INLINE`: dispatch returns the terminal logical result in the same interaction.
- `DEFERRED`: dispatch returns accepted/existing work plus an opaque handle; terminal completion is observed later.

For `DEFERRED`, a binding declares one or more supported observation methods:

```text
CALLBACK
POLL
EVENT
WATCH
```

Observation methods are protocol capabilities, not task kinds. Their exact wire protocol remains for a later review.

## Illustrative bindings

The examples below are conceptual records, not YAML schemas.

```text
Binding decoder-production revision 17
  capability = data.bag.decode@v3
  provider = SERVICE
  completion = DEFERRED
  adapter = service
  target = decoder.production
  observe = CALLBACK + POLL
  cancel = BEST_EFFORT
  admission-policy = decoder.production revision 4
```

The same logical capability in another environment could use:

```text
Binding decoder-worker revision 23
  capability = data.bag.decode@v3
  provider = WORKER
  completion = DEFERRED
  adapter = worker
  target = decoder.worker.v2
  observe = EVENT
  cancel = BEST_EFFORT
```

Both bindings satisfy the same TaskDefinition. Neither changes the business DAG.

A GPU-backed encoder could use:

```text
Binding encoder-compute revision 8
  capability = data.embedding.encode@v2
  provider = COMPUTE
  completion = DEFERRED
  adapter = compute
  target = embedding.encoder.v2
  observe = POLL + WATCH
  resource-intent = accelerator class GPU, count 1
```

Alibaba, Volcano, Kubernetes, Slurm, or another platform may realize that compute request behind the adapter. Those names and native objects are not part of WorkflowSpec, TaskSpec, or TaskDefinition.

## Resolution and snapshot pinning

Binding selection occurs after logical contract validation and before dispatch:

```text
WorkflowSpec + exact TaskDefinitions
               │
               ▼
resolve capabilities for target environment
               │
               ▼
pin ExecutionBinding revisions in ExecutionSnapshot
               │
               ▼
create/dispatch TaskAttempts
```

An active run never follows mutable binding configuration. Operational routing behind a stable ExecutionTarget may change physical instances, but not provider/completion semantics or logical contract guarantees.

If the requested environment has no compatible binding, submission/snapshot creation fails before any task dispatch.

## AdmissionPolicy is separate

An `AdmissionPolicy` protects a logical downstream capability or target, for example:

```text
AdmissionPolicy decoder.production revision 4
  max-active-attempts = 200
```

The binding references the policy; the limit is not part of TaskDefinition and does not describe physical hardware. Future tenant, priority, workflow-class, or rate-limit dimensions remain out of scope for this review.

## ResourceIntent is separate

`ResourceIntent` expresses a physical execution request passed to the Compute Platform. It does not grant capacity and does not replace logical admission.

```text
dependency-ready
      │
      ▼
Andormu AdmissionPolicy
      │ admitted
      ▼
TaskExecutionRequest + ResourceIntent
      │
      ▼
Compute Platform physical admission/placement
```

Do not combine these as “GPU slots”. A decoder concurrency limit and a compute request for GPUs are different decisions, owned by different layers.

## Validation rules for this review

Before a binding can be pinned:

- capability must exactly match the TaskDefinition requirement;
- deferred completion must declare at least one observation method;
- inline completion must not require later observation for normal success;
- adapter, provider, completion model, and target must form a supported combination;
- ResourceIntent is valid only where the selected realization can pass it to a physical resource authority;
- referenced AdmissionPolicy and binding revisions must be resolvable and immutable for the snapshot.

Exact enums, defaults, compatibility negotiation, callback authentication, and failure codes remain for the unified execution-protocol review.

## Non-goals

This object does not:

- define DAG nodes or dependencies;
- choose the domain workflow;
- deploy or scale services;
- expose transient physical endpoints to the workflow;
- allocate resources;
- make backend-native state canonical;
- define the TaskAttempt state machine.
