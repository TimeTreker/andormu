# State Model

The proposed model separates **simple lifecycle states** from **detailed reason/failure descriptors**.

## WorkflowRun states

```text
PENDING ──► RUNNING ───────────────► SUCCEEDED
              │  │  │
              │  │  ├──────────────► FAILED
              │  │  ├──────────────► TIMED_OUT
              │  │  └─► CANCELLING ─► CANCELLED
              │  └────► SUSPENDED ───► RUNNING
              └──────────────────────► TERMINATED  (emergency operation)
```

`RECOVERING` is intentionally not required as a primary state. A redrive can append a `RedriveAttempt` and move a failed run back to `RUNNING` under explicit rules.

## TaskRun states

Proposed minimal set:

- `BLOCKED` — dependency conditions not yet satisfied.
- `READY` — eligible to create/dispatch an attempt.
- `DISPATCHING` — an attempt exists and dispatch is being established.
- `RUNNING` — current attempt is executing.
- `RETRY_WAIT` — failed attempt has a future retry time.
- `SUCCEEDED` — terminal success.
- `FAILED` — terminal failure, no retry remains.
- `TIMED_OUT` — terminal timeout.
- `CANCELLED` — terminal cancellation.
- `SKIPPED` — terminal non-execution because branch/dependency rules selected another path.

## TaskAttempt states

- `CREATED`
- `DISPATCHED`
- `RUNNING`
- `SUCCEEDED`
- `FAILED`
- `TIMED_OUT`
- `CANCELLED`
- `LOST`

`LOST` describes an attempt whose backend execution can no longer be reliably observed. The TaskRun retry/failure policy decides what follows.

## State reason

Each state should carry a structured reason code/message rather than creating dozens of new enum states. Examples:

- `DEPENDENCIES_PENDING`
- `RETRY_BACKOFF`
- `RESOURCE_ADMISSION_PENDING`
- `BRANCH_NOT_SELECTED`
- `UPSTREAM_REQUIRED_FAILURE`
- `BACKEND_EVICTED`
- `USER_CANCELLED`
