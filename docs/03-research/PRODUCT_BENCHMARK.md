# Workflow Engine Product Benchmark

Research refreshed: **2026-09-02**.

| System | Strong design to absorb | Do not copy into Andormu core |
|---|---|---|
| Temporal | Durable Event History; crash recovery; explicit Activity retry/cancellation; stable workflow identity | Deterministic user workflow-code replay requirement; Andormu uses declarative specs instead |
| Flyte / Propeller | Desired-state reconciliation; DAG/node executor split; dynamic nodes; recovery that reuses successful nodes | Kubernetes CRD as canonical public contract; physical task execution coupling |
| Argo Workflows | DAG execution; retry/backoff; suspend/resume; exit handlers; memoization/work avoidance | Kubernetes Pod fields and scheduling concepts in canonical task model |
| Airflow 3 | DAG/Task/DagRun/TaskInstance distinction; trigger rules; setup/teardown; task history | Cron/data-interval semantics as core; executor/resource coupling |
| Prefect 3 | Rich run-state visibility; state reasons; events/automations; cached/retry views | Python-first workflow execution model as wire contract |
| Tekton | PipelineRun/TaskRun distinction; finally tasks; graceful stop/cancel; pipeline/task/finally timeouts | Kubernetes-native Task/Pod public API |
| AWS Step Functions | Retry/Catch; redrive preserving successful states and pinned definition | Cloud/provider-specific service integration state language |
| Conductor OSS / Orkes | Service/worker orchestration; HTTP/event/wait tasks; dynamic fork/join; reusable TaskDefinition; at-least-once worker delivery; recovery operations | Monolithic system-task catalog and service-specific concerns in core |
| Hatchet | Durable task queue + DAG; worker slots/capacity; high-volume parallel task dispatch; event history | Do not inherit product/database assumptions before semantic-fit review |
| Kestra 2 | Controller/worker separation; remote workers; reduced payload movement through the control plane | Do not adopt newly evolving engine internals wholesale |
| Open Workflow Specification | Vendor-neutral workflow concepts; fork/switch/try/wait/listen/run; CloudEvents interoperability | Adopt wholesale as canonical format before validating DAG/resource/artifact needs |
| CloudEvents | Portable event envelope | Does not define Andormu state semantics |
| OpenLineage | Standard run/job/dataset lineage export | Run state model is too small to replace internal workflow execution state |

## Key conclusion

The best fit for Andormu is a **declarative DAG + reconciliation engine with durable execution history and service/worker-aware dispatch**, not a code-replay workflow runtime and not a Kubernetes-native pipeline CRD.

For a future implementation substrate, **Conductor OSS and Hatchet deserve the first semantic-fit benchmark** because both naturally support durable service/worker task execution. This is not yet a decision to fork or embed either engine.
