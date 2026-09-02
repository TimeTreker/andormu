# Idempotency and Delivery Semantics

## Reality: external delivery is at least once

Network failures can make Andormu unsure whether a dispatch/cancel callback succeeded. Therefore control messages must be safely repeatable.

## Required identities

- WorkflowRun id
- TaskRun id
- TaskAttempt id
- command/idempotency key
- event id

## Dispatch rule

Repeated `DispatchTaskAttempt(attempt_id=X)` must refer to the **same logical attempt**. The adapter/backend is responsible for deduplicating the stable attempt id or returning the existing execution handle.

A retry uses a new attempt id.

## API commands

Mutating external APIs should accept idempotency keys where feasible, especially:

- submit workflow,
- cancel,
- suspend/resume,
- redrive,
- adapter dispatch/cancel.

## External side effects

Andormu cannot make an arbitrary payment/API write/file mutation exactly once. Task owners must use one of:

- idempotent side effects,
- externally supported idempotency keys,
- compensation/cleanup logic,
- non-retryable policies when repetition is unsafe.

## Events

External event delivery should be at least once. Consumers deduplicate by event id.
