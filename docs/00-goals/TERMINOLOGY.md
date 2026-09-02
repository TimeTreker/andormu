# Terminology

| Term | Meaning |
|---|---|
| WorkflowDefinition | Reusable logical workflow authored/generated before execution. |
| WorkflowRevision | Immutable version of a WorkflowDefinition. |
| WorkflowSpec | Canonical declarative graph submitted to Andormu. A revision may contain or resolve to a WorkflowSpec. |
| ExecutionSnapshot | Fully resolved immutable snapshot pinned by one WorkflowRun. |
| WorkflowRun | One logical execution of an ExecutionSnapshot. |
| NodeSpec | One graph node in a WorkflowSpec. |
| TaskDefinition | Reusable versioned executable-task contract. |
| TaskSpec | One task occurrence inside a workflow; binds a TaskDefinition/executor plus inputs, policies, dependencies, and resource intent. |
| TaskRun | One logical execution of a TaskSpec within a WorkflowRun. |
| TaskAttempt | One concrete execution attempt of a TaskRun. Retries create additional attempts. |
| DependencyRule | Predicate determining when a downstream node becomes runnable or skipped. |
| Finalizer | Explicit cleanup/post-run node(s) evaluated under dedicated finalization semantics. |
| ResourceIntent | Resource requirement passed through to the Compute Platform; not an allocation. |
| ExecutionHandle | Opaque handle returned by an execution adapter for one TaskAttempt. |
| Redrive | Continue the same failed WorkflowRun from failed/unreached work using the same ExecutionSnapshot. |
| Restart | Create a new WorkflowRun from the beginning using a selected snapshot/revision. |
| Reconcile | Re-evaluate desired graph state against durable observed state and issue only required next commands. |
| DynamicExpansion | Explicit runtime creation of additional acyclic node instances from a recorded expansion result. |
