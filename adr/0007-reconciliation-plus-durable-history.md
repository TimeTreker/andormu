# ADR-0007: Reconciliation plus durable execution history

**Status:** Proposed

The engine should reconcile immutable desired workflow state against durable observed execution state. Lifecycle transitions/operator actions are append-only and queryable. The implementation may use materialized state plus an event journal rather than Temporal-style replay of user code.
