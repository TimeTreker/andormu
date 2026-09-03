# Asynchronous Task Protocol

## Problem

Many useful tasks are longer than a safe synchronous RPC lifetime. Examples include video decoding, point-cloud conversion, large log parsing, or remote compute submission.

Andormu therefore needs a durable deferred-completion protocol. `DEFERRED` is selected by an environment ExecutionBinding and is independent of whether the provider is a service, worker, compute platform, or external runtime.

## Proposed lifecycle

```text
TaskAttempt CREATED
      │
      ▼
DISPATCHED / SUBMITTED
      │
      ▼
WAITING / RUNNING
      │
      ├── callback/event
      ├── observe/poll
      └── heartbeat/lease (optional)
      │
      ▼
SUCCEEDED | FAILED | CANCELLED | LOST
```

Exact canonical state names remain a Phase-0 review item; the important invariant is that submission and completion are durably correlated by the stable `TaskAttempt` identity.

## Submit response

A successful asynchronous submit returns or resolves an opaque `ExecutionHandle`.

The handle is backend-specific and must not leak into dependency semantics.

## Completion notification

An ExecutionBinding with `completion = DEFERRED` declares one or more supported observation methods:

- callback endpoint,
- CloudEvents-compatible event,
- durable message/queue event,
- polling through `Observe`,
- backend-native watch stream.

All notifications must be idempotently correlated to the same TaskAttempt.

## Duplicate delivery

At-least-once transport is assumed.

Duplicate submit/observe/completion messages for one TaskAttempt must not create another logical attempt or duplicate terminal transitions.

## Lost backend

If an execution handle can no longer be observed, Andormu must not silently invent success/failure. The adapter should report a structured `LOST`/unknown execution condition that is classified by policy into:

- recover observation,
- retry with a new TaskAttempt,
- fail TaskRun,
- require operator decision.

## Cancellation

Cancel is best effort across distributed service boundaries. Andormu records cancellation intent durably before issuing backend cancellation and continues observing until a defined terminal/cancellation outcome is reached.
