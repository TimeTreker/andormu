# Retry and Timeout Semantics

## Retry belongs to TaskRun; attempts are immutable

A failed attempt can schedule a future attempt under one TaskRun.

## Proposed RetryPolicy

- maximum attempts,
- initial delay,
- backoff type: fixed / linear / exponential,
- multiplier,
- maximum delay,
- jitter,
- retryable failure classes/codes,
- non-retryable failure classes/codes,
- optional total retry budget.

Jitter is important to avoid synchronized retry storms.

## Failure classification

Retry decisions should use structured failure data, not string matching. Proposed classes:

- `WORKLOAD`
- `TRANSIENT_EXTERNAL`
- `INFRASTRUCTURE`
- `RESOURCE`
- `TIMEOUT`
- `ENGINE`
- `CANCELLED`
- `UNKNOWN`

A backend may provide a retryable hint, but Andormu policy remains authoritative.

## Timeouts

Phase 0 should distinguish at least:

- **workflow timeout** — total WorkflowRun wall-clock budget,
- **task total timeout** — TaskRun budget across retries,
- **attempt start/dispatch timeout** — attempt cannot become running in time,
- **attempt execution timeout** — running attempt exceeds budget,
- **heartbeat/liveness timeout** — optional for adapters that support heartbeats.

Exact treatment of suspended time is an open design question and must be resolved before schemas are frozen.
