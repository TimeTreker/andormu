# Terminology

| Term | Meaning |
|---|---|
| Timeways | Bronze Dragonflight's vendor-neutral execution platform. It combines Andormu's durable control-plane semantics with standard service, data-processing, compute, messaging, storage, and observability infrastructure. |
| Andormu | The durable control plane of Timeways. It owns canonical workflow/task orchestration semantics, logical admission, durable execution state, and recovery. |
| Execution Plane | The heterogeneous external systems that perform actual work: services, workers, data engines, compute runtimes, and cloud/on-prem platforms. These systems are not reimplemented by Andormu. |
| WorkflowDefinition | Reusable logical workflow authored/generated before execution. |
| WorkflowRevision | Immutable version of a WorkflowDefinition. |
| WorkflowSpec | Canonical Bronze declarative graph submitted to Andormu. A revision may contain or resolve to a WorkflowSpec. It must not require vendor-native execution objects. |
| ExecutionSnapshot | Proposed fully resolved immutable snapshot pinned by one WorkflowRun. |
| WorkflowRun | One logical execution of an ExecutionSnapshot. |
| NodeSpec | One graph-semantic node in a WorkflowSpec. Node kind describes logical graph behavior, not runtime/deployment mechanism. |
| TaskDefinition | Reusable versioned logical executable-task contract. |
| TaskSpec | One task occurrence inside a workflow; binds logical capability/interface plus inputs, policies, dependencies, and optional resource intent. |
| TaskRun | Proposed logical execution of a TaskSpec within a WorkflowRun. |
| TaskAttempt | Proposed concrete attempt to fulfill a TaskRun. Its execution may be realized by a persistent service, deferred operation, worker, data engine, external workflow/job, or compute job. |
| ExecutionRealization | The adapter/backend mechanism used to fulfill one TaskAttempt. It is separate from DAG topology and logical Task meaning. |
| InlineCompletion | An execution completion model where dispatch returns the terminal task result in the same interaction. |
| DeferredCompletion | An execution completion model where dispatch establishes accepted work/handle and terminal completion is learned later by observe, callback, event, watch, or equivalent mechanism. |
| LogicalAdmission | Andormu decision that logically ready work may proceed to attempt creation/dispatch under workflow/capability/backpressure policy. It is distinct from physical resource admission. |
| PhysicalAdmission | Compute/cloud/platform decision about quota, queue, accelerator/CPU availability, placement, topology, reservation, or preemption. |
| DependencyRule | Predicate determining when a downstream node becomes runnable or skipped. |
| Finalizer | Explicit cleanup/post-run node(s) evaluated under dedicated finalization semantics. |
| ResourceIntent | Resource requirement passed through to the Compute Platform; not an allocation. |
| ExecutionHandle | Opaque handle returned by an execution adapter for one TaskAttempt. |
| Redrive | Continue the same failed WorkflowRun from failed/unreached work using the same ExecutionSnapshot. |
| Restart | Create a new WorkflowRun from the beginning using a selected snapshot/revision. |
| Reconcile | Re-evaluate desired graph state against durable observed state and issue only required next commands. |
| DynamicExpansion | Explicit runtime creation of additional acyclic node instances from a recorded expansion result. |
