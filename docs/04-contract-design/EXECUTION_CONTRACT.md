# Task Execution Adapter Contract

Andormu should communicate with execution adapters through a small generic protocol.

## Commands

- Dispatch TaskAttempt
- Observe/Get TaskAttempt
- Cancel TaskAttempt
- Optional heartbeat/renew if backend requires it

## Dispatch request contains

- stable TaskAttempt id,
- task execution descriptor/reference,
- resolved input/value/artifact refs,
- resource intent,
- timeout/cancellation metadata,
- trace/correlation context,
- secret references,
- idempotency key.

## Adapter response

- accepted/existing status,
- opaque ExecutionHandle,
- current generic phase,
- backend references/log links,
- structured failure if rejected.

## Important invariant

Repeated dispatch for one TaskAttempt id must not create a second logical attempt.
