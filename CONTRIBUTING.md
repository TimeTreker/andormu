# Contributing during Phase 0

Andormu is currently in **design-first Phase 0**.

## Allowed contributions

- Goal and non-goal clarification.
- Architecture diagrams and state-machine proposals.
- Product workflows and operational UX.
- Contract proposals in Markdown.
- ADRs and design questions.
- Competitive research with primary-source references.
- Review checklists and acceptance criteria.

## Not allowed before the Design Gate

- Runtime implementation code.
- API server implementation.
- Database migrations.
- Kubernetes controllers/operators.
- Generated SDKs.
- Executable JSON/Proto schemas presented as final contracts.
- CI that builds/releases product binaries.

The `coding/` directory exists only to capture future work without prematurely implementing it.

## Design changes

Semantic changes should update:

1. The relevant architecture/product document.
2. Any affected contract-design document.
3. An ADR if the change establishes or overturns a durable decision.
4. `review/PHASE0_REVIEW.md` if it changes the Design Gate.
