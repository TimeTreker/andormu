# Contract Design — Phase 0

These documents define semantic intent only. **Do not treat them as frozen JSON/Proto schemas yet.**

Proposed contract families:

- WorkflowSpec / WorkflowRevision
- ExecutionSnapshot
- TaskDefinition / TaskSpec
- WorkflowRun / TaskRun / TaskAttempt
- RetryPolicy / TimeoutPolicy
- FailureDescriptor
- ResourceIntent reference/pass-through
- TaskExecutionRequest / ExecutionObservation
- WorkflowEvent
- Operator commands (suspend/cancel/redrive/etc.)

After the Design Gate, normative shared contracts should be coordinated with the `bronze-dragonflight` architecture repository.
