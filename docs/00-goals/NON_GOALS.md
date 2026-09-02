# Non-goals

Andormu is intentionally narrower than a full MLOps, data, or cluster platform.

## NG1 — No domain semantics

Andormu does not understand datasets, training epochs, model checkpoints, evaluation suites, or simulation scenarios.

## NG2 — No physical resource scheduler

Andormu does not choose nodes, GPUs, NUMA domains, queues, quotas, placements, or preemption victims. It may carry a `ResourceIntent` to the Compute Platform.

## NG3 — No Kubernetes-first public model

Kubernetes Pods, Jobs, CRDs, affinities, tolerations, and namespaces must not leak into the canonical Andormu domain contract.

## NG4 — No user business logic runtime in core

Task implementations live in external runtimes/workers/services. Andormu dispatches and observes them.

## NG5 — No promise of exactly-once side effects

Distributed workflow engines cannot make an arbitrary external side effect exactly once. Andormu must instead provide stable attempt identities, idempotent control APIs, and explicit retry/recovery semantics.

## NG6 — No artifact warehouse

Large task inputs/outputs should be passed by reference. Andormu may retain small metadata and references, not become the dataset/model/object-storage system.

## NG7 — No generic lineage platform

Execution relationships are tracked, but broad data/model lineage remains owned by the relevant metadata platforms. OpenLineage export can be an integration.

## NG8 — No visual low-code authoring requirement in Phase 1

The primary producers of `WorkflowSpec` are domain platforms and SDK/contract clients. Phase-1 UI prioritizes validation, observation, debugging, and recovery.

## NG9 — No cron-first scheduler requirement in Phase 1

Recurring schedules can be created by callers or a future scheduling service. Andormu's core responsibility begins with a submitted workflow run.

## NG10 — No arbitrary cyclic graph

The canonical execution graph remains acyclic. Iteration is represented as bounded/dynamic expansion into unique node instances, not graph cycles.


## NG11 — No service lifecycle platform

Andormu does not deploy services, manage replica counts, perform service discovery infrastructure, or own cluster autoscaling. It orchestrates logical work against execution targets.

## NG12 — No domain workflow selector

Andormu does not inspect file extensions, dataset types, model families, or other domain metadata to decide which business workflow should run. The caller/domain platform owns workflow selection or compilation.
