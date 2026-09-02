# Workflow Contract Proposal

## Required semantic fields

- contract/api version,
- workflow identity/revision,
- interface (inputs/outputs),
- node ids and node kinds,
- dependency ids + trigger rules,
- task/subworkflow references,
- workflow policies,
- finalizers,
- annotations.

## Submission result

The engine returns:

- WorkflowRun id,
- resolved ExecutionSnapshot id/digest,
- accepted schema version,
- validation warnings.

## Immutability

After activation, the run references a pinned snapshot. A definition update never mutates historical/running snapshots.
