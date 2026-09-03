# ADR-0018: DAG task semantics are separate from execution realization

**Status:** Accepted

## Context

The same logical task may be fulfilled today by an HTTP service, tomorrow by a worker pool, and later by a Flink/Spark or compute job.

Encoding `service`, `async_service`, `kubernetes_job`, or similar deployment mechanisms as business DAG node kinds would couple workflow topology to transient implementation choices.

The term “async service” also conflates two different dimensions: persistent executor lifecycle and deferred task completion.

## Decision

Canonical DAG node kinds describe logical graph semantics.

A canonical `Task` describes logical work. A `TaskAttempt` is fulfilled through an **execution realization** behind an adapter.

Execution realizations may include:

- persistent service with inline completion;
- persistent service with deferred completion;
- worker/task queue;
- Flink/Spark/data-engine job;
- external workflow/runtime operation;
- compute-backed job.

Persistent service lifecycle and inline/deferred completion are separate dimensions.

DAG dependency evaluation uses logical task outcomes, not backend-native phases.

## Consequences

- Runtime/provider changes need not change business DAG topology when the logical contract remains compatible.
- Execution adapters must normalize backend submission/observation/cancellation.
- If another mature engine owns an internal DAG, Andormu normally treats that execution as one opaque TaskAttempt rather than duplicating its internal graph/state machine.
