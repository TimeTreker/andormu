# Problem Statement

Bronze Dragonflight will contain multiple domain platforms that all need workflow execution:

- Zidormi needs data-processing workflows.
- Anachronos needs training workflows.
- Evaluation needs benchmark workflows.
- Simulation needs scenario/execution/metric workflows.

If each platform implements its own DAG runner, the ecosystem will duplicate:

- dependency scheduling,
- retries and backoff,
- cancellation semantics,
- workflow persistence,
- task state machines,
- recovery logic,
- event history,
- observability,
- and backend integration.

If instead one domain platform owns the workflow engine, every other platform becomes coupled to that domain.

Andormu therefore exists as a **shared workflow execution plane** with a narrow contract:

```text
Domain Intent
    ↓ compiled by domain platform
WorkflowSpec
    ↓
Andormu
    ↓
TaskExecutionRequest
    ↓
Execution / Compute ecosystem
```

The core design challenge is not drawing DAGs. It is defining reliable semantics around state, failure, retries, cancellation, dynamic expansion, idempotency, and recovery while preserving clean ownership boundaries.
