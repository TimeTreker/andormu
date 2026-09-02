# Personas and Jobs-to-be-Done

## Domain platform developer

**Examples:** Anachronos or Zidormi developer.

Needs to:

- compile domain intent into a stable `WorkflowSpec`,
- submit a workflow and receive a `WorkflowRunId`,
- get deterministic validation errors,
- observe task/run status through APIs/events,
- avoid depending on Kubernetes/Slurm details.

## Workflow operator / SRE

Needs to:

- identify blocked or failing tasks quickly,
- distinguish workload failure from infrastructure loss,
- inspect attempts, retries, backend handles, logs, and events,
- suspend, resume, cancel, terminate, or redrive safely,
- understand what cleanup/finalizer work will still run.

## Platform architect

Needs to:

- evolve contracts without breaking domain platforms,
- reason about ownership boundaries,
- audit execution history,
- support multiple runtimes and clusters without changing workflow semantics.

## Security / governance operator

Needs to:

- know who submitted/modified/cancelled/redrove work,
- constrain permissions by namespace/workspace,
- prevent inline secret leakage,
- preserve immutable revision and audit records.
