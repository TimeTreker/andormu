# Workflow Contract Proposal

This document records semantic requirements only. It does not define a finalized JSON, YAML, or Proto schema.

## Contract boundary

A WorkflowSpec is the canonical Bronze declarative business graph. It contains logical task/control nodes and their data/dependency relationships. Environment-specific execution realization is resolved through `ExecutionBinding` and pinned in `ExecutionSnapshot`; it is not embedded in the graph.

## Required semantic fields

- contract/API version;
- workflow identity and immutable revision/content digest;
- declared input/output interface;
- node ids and logical node kinds;
- exact TaskDefinition or subworkflow revision references;
- input/output bindings;
- dependency ids and trigger rules;
- logical workflow/task policy bindings and permitted overrides;
- finalizers;
- bounded annotations.

The canonical contract must reject provider kinds, adapter references, endpoints, transport configuration, backend-native objects, physical ResourceIntent, and environment AdmissionPolicy values inside TaskSpec.

## Submission request semantics

A submission provides:

- an exact WorkflowRevision or an ad-hoc WorkflowSpec to be content-addressed;
- bound workflow inputs, including ArtifactRefs rather than large payloads;
- target environment/resolution context;
- a caller-defined idempotency key and scope;
- trace/audit context.

## Submission idempotency

Within the declared caller/scope:

- the first accepted key creates one WorkflowRun;
- a retry with the same key and semantically equivalent request returns that existing run;
- reuse of the key with a materially different workflow revision, inputs, resolution context, or relevant policy identity is rejected as a conflict;
- retention and expiry of deduplication records remain to be decided before schema freeze.

Workflow-submission idempotency does not replace TaskAttempt dispatch idempotency. A task retry always creates a new attempt identity.

## Validation before activation

Andormu validates at least:

- graph structure, acyclicity, node/dependency references, and interface bindings;
- exact TaskDefinition revision availability;
- input/output type compatibility, including ArtifactRef positions;
- policy placement and override rules;
- one compatible ExecutionBinding revision for every required capability in the selected environment.

No task is dispatched until resolution succeeds and the immutable ExecutionSnapshot is durably established.

## Submission result

An accepted submission returns:

- WorkflowRun id;
- resolved ExecutionSnapshot id/digest;
- accepted contract version;
- whether the request created or reused the run;
- validation warnings.

## Snapshot immutability

The snapshot pins the WorkflowSpec/WorkflowRevision, all TaskDefinition revisions, all resolved ExecutionBinding revisions, applicable policy revisions, and bound inputs. Definition, binding, or policy updates never mutate active or historical snapshots.
