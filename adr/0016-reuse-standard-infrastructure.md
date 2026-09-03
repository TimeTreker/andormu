# ADR-0016: Reuse standardized infrastructure; own orchestration semantics

**Status:** Accepted

## Context

Messaging, stream/batch processing, container orchestration, compute scheduling, GPU virtualization, storage, databases, service discovery, and observability are mature infrastructure domains.

Reimplementing them inside Andormu would create unnecessary scope, operational risk, and competing authorities.

## Decision

Andormu follows the principle:

> **Own the semantics; reuse the infrastructure.**

Andormu may depend on mature infrastructure capabilities and integrate with cloud/on-prem products, but it does not implement replacements for Kafka/MQ, Flink/Spark, Kubernetes/container runtimes, Slurm/Ray, GPU virtualization/scheduling, object storage, databases, service discovery, or observability systems.

These systems must not become the canonical Andormu workflow semantics.

## Consequences

- Production dependencies are expected and healthy when their capability boundary is explicit.
- Kafka delivery identity is not TaskAttempt identity.
- Kubernetes/Flink/cloud native states are normalized through adapters rather than becoming TaskRun states directly.
- Infrastructure abstractions should be introduced only where they protect an Andormu semantic boundary; do not wrap every standard API for its own sake.
