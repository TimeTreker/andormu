# Operations and Recovery UX

## Suspend

Preview: no new task attempts will dispatch; currently running tasks continue under the proposed default semantics.

## Cancel

Preview:

- active attempts that will receive cancel requests,
- pending work that will not start,
- finalizers that will run.

## Terminate

Dangerous action with stronger permission and confirmation. Explain that finalizers are not guaranteed.

## Redrive

For a failed/timed-out run, preview:

- ExecutionSnapshot digest (unchanged),
- completed TaskRuns that will be preserved,
- failed/unreached work to be reactivated,
- retry-budget handling,
- side-effect warning.

## Restart

Creates a new WorkflowRun. UI must clearly distinguish restart from redrive.
