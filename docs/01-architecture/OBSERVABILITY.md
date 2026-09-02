# Observability

Andormu owns workflow-semantic observability, not every backend metric.

## Native metrics

Examples:

- active WorkflowRuns,
- TaskRuns by state,
- reconcile latency,
- ready-to-dispatch latency,
- task transition latency,
- retry counts,
- redrive counts,
- dispatch errors,
- lost attempts,
- finalizer failures,
- event publication lag.

## Tracing

Every workflow, task run, and attempt should carry correlation/trace context through adapters when possible.

## Logs

Andormu may retain its own control-plane logs and references/links to task logs. The underlying log storage remains external.

## Explainability

The product must be able to answer “why is this not running?” using dependency state, suspension state, logical concurrency, adapter/admission status, retry backoff, and failure evidence.
