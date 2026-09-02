# TaskSpec Design

This is a design proposal, not a normative schema.

**Core rule:** a task is a logical unit of work, not a process/container/Pod. See `SERVICE_TASK_EXECUTION.md` and ADR-0013.

## TaskDefinition

Reusable properties may include:

- stable name + revision,
- executor/runtime type,
- capability/execution-target reference (service capability, worker type, runtime template, plugin reference, or compute job descriptor),
- input/output interface,
- default retry/timeout policy,
- capability metadata,
- observability/log-link capabilities.

## TaskSpec

One workflow occurrence adds:

- node id,
- exact TaskDefinition reference or inline execution descriptor,
- input bindings,
- output bindings,
- dependencies,
- trigger rule,
- retry policy overrides,
- timeout overrides,
- `ResourceIntent`,
- idempotency metadata,
- finalizer/cleanup classification when applicable,
- labels/annotations.

## Attempt identity

Every dispatched attempt receives a stable globally unique attempt id. Repeated delivery of the same dispatch command must carry the same attempt id. A retry creates a **new** attempt id under the same TaskRun.

## Backend handle

`ExecutionHandle` is opaque to core scheduling logic. The adapter can use it to observe/cancel the external execution.
