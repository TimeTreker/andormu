# Cancellation, Suspension, and Termination

Different operational intents need different commands.

## Suspend

Intent: **pause workflow scheduling without declaring failure**.

Proposed behavior:

- no new normal TaskAttempts are dispatched,
- already-running attempts are not automatically killed,
- run remains resumable,
- timeout treatment during suspension is still an open question.

## Resume

Reactivates reconciliation using the same ExecutionSnapshot and durable state.

## Cancel

Intent: **gracefully stop the workflow**.

Proposed behavior:

- no new normal tasks,
- request cancellation of active cancellable attempts,
- run explicit finalizers according to policy,
- terminal result `CANCELLED`.

## Terminate

Intent: **emergency hard stop**.

Proposed behavior:

- best-effort cancellation of active attempts,
- finalizers are not guaranteed,
- terminal result `TERMINATED`,
- requires stronger permission and audit reason.

## Why separate these

Tekton and Argo expose important distinctions between graceful stop/cancel and hard termination. Collapsing them into one “stop” button produces unsafe cleanup behavior.
