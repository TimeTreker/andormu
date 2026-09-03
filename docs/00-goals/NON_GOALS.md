# Non-goals

Andormu is intentionally narrower than a full MLOps, data, compute, or cluster platform. It is the durable control plane of Timeways, not a replacement for mature execution infrastructure.

## NG1 — No domain semantics

Andormu does not understand datasets, training epochs, model checkpoints, evaluation suites, simulation scenarios, bag-file meaning, or normal-file business policy.

## NG2 — No physical resource scheduler

Andormu does not choose nodes, GPUs, NUMA domains, physical queues, quotas, placements, topology, reservations, or preemption victims. It may carry a `ResourceIntent` to the Compute Platform.

## NG3 — No Kubernetes/cloud-first public model

Kubernetes Pods, Jobs, CRDs, affinities, tolerations, Slurm jobs, Ray jobs, cloud workflow objects, and vendor-specific execution objects must not leak into the canonical Bronze/Andormu workflow contract.

## NG4 — No user business logic runtime in core

Task implementations live in external runtimes/workers/services/data engines. Andormu dispatches and observes them.

## NG5 — No promise of exactly-once side effects

Distributed workflow engines cannot make an arbitrary external side effect exactly once. Andormu must instead provide stable attempt identities, idempotent control APIs, and explicit retry/recovery semantics.

## NG6 — No artifact warehouse

Large task inputs/outputs should be passed by reference. Andormu may retain small metadata and references, not become the dataset/model/object-storage system.

## NG7 — No generic lineage platform

Execution relationships are tracked, but broad data/model lineage remains owned by the relevant metadata platforms. OpenLineage export can be an integration.

## NG8 — No visual low-code authoring requirement in Phase 1

The primary producers of `WorkflowSpec` are domain platforms and SDK/contract clients. Phase-1 UI prioritizes validation, observation, debugging, admission visibility, and recovery.

## NG9 — No cron-first scheduler requirement in Phase 1

Recurring schedules can be created by callers or a future scheduling service. Andormu's core responsibility begins with a submitted workflow run.

## NG10 — No arbitrary cyclic graph

The canonical execution graph remains acyclic. Iteration is represented as bounded/dynamic expansion into unique node instances, not graph cycles.

## NG11 — No service lifecycle platform

Andormu does not deploy services, manage replica counts, perform service discovery infrastructure, or own cluster autoscaling. It orchestrates logical work against execution targets.

## NG12 — No domain workflow selector

Andormu does not inspect file extensions, dataset types, model families, or other domain metadata to decide which business workflow should run. The caller/domain platform owns workflow selection or compilation.

## NG13 — No reimplementation of standardized infrastructure

Andormu does not implement replacements for:

- Kafka, RocketMQ, Pulsar, or general message brokers;
- Flink, Spark, or general stream/batch data-processing engines;
- Kubernetes, container runtimes, Slurm, Ray, or general cluster/compute execution systems;
- CUDA runtime, device plugins, MIG, vGPU, MPS, or GPU virtualization/scheduling infrastructure;
- OSS/S3/TOS/COS or general object/file storage;
- PostgreSQL/MySQL/Redis or general database/cache systems;
- service discovery, networking, secret stores, metrics, tracing, or logging platforms.

These systems may be production dependencies, execution backends, or integration points.

## NG14 — No takeover of another engine's internal DAG

If Flink, Spark, Ray, a training runtime, a cloud workflow service, or another mature system already owns an internal execution graph, Andormu normally treats that execution as one opaque TaskAttempt.

Andormu does not mirror every internal operator/job as a second competing orchestration graph unless a future explicit contract intentionally exposes those operations as independent Bronze tasks.

## NG15 — No vendor workflow contract as canonical Bronze contract

Alibaba CloudFlow, Argo Workflow, cloud ML workflow YAML, Kubernetes CRDs, and similar product contracts may be integration targets, but they do not define Bronze Dragonflight workflow semantics.
