# Security and Governance

## Namespaces/workspaces

Every WorkflowDefinition and WorkflowRun belongs to an ownership scope used for authorization and query isolation.

## RBAC actions

At minimum distinguish:

- view definitions/runs,
- submit,
- suspend/resume,
- cancel,
- terminate,
- redrive/restart,
- manage task definitions,
- manage workflow revisions,
- administrative override.

## Audit

Mutating actions record:

- principal,
- timestamp,
- action,
- target,
- reason/comment when required,
- idempotency key/request id,
- before/after semantic revision if applicable.

## Secrets

Workflow specs contain secret references only. Secret resolution occurs in an authorized execution layer.

## Supply-chain metadata

TaskDefinition revisions should be able to reference immutable executable digests (container digest, package digest, etc.) rather than mutable tags.
